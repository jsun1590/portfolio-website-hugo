---
title: "idekCTF 2022: Osint Crime Confusion 1: W as in Where"
date: 2023-01-16T17:37:37+08:00
draft: false
author: "Jack Sun"
tags:
  - Cybersecurity
  - CTFs
  - Writeups
  - idekCTF
  - idekCTF 2022
  - Osint Crime Confusion
  - OSINT
image: /images/writeups/idekCTF2022/banner.png
---

This is a 4 part OSINT challenge centered around the theme of investigating a murder. During the CTF, I solved parts 2 to 4 while one of my teammates (`toasterpwn`) solved part 1.

You can read the other parts of this challenge [here](https://jacksun.dev/tags/osint-crime-confusion/)

## Description

- Name: Osint Crime Confusion 1: W as in Where
- Category: OSINT
- 462 pts, 74 solves
- Written by `Franfrancisco9#0105`

Someone has died unexpectedly. The police is on it, but between you and me, I cannot wait for the police. I am a private investigator and I need your help. Unfortunately, we might be tracked so I cannot give you the information directly. Start in a major social network. Certainly not a problem for the best hacker I know right...? Alright here goes a beautiful poem:

```text
Some people in weird ways were connected
Some were a triangle, some were less directed
For one night they all met
At doctor's Jonathan Abigdail the third they wept 
Things were said, threats in the air
A few days later someone is dead
Who is that someone? That is for you to find,
Also who is the killer, if you really don't mind
```

*Note for the all the challenges:* The challenge is divided into three challenges: Where, Weapon, and Who. Where is the first one the others will come later. In each one you can find the flag somewhere online. You might find the information in any order, however the expected order is: Where, Weapon and Who. Example: If the answer is knife then when you would discover that somewhere: like "the killer used a `idek{knife_V5478G}`" or instructions on how to get the flag: like `idek{weaponUsed_V5478G}`. The flag would then be `idek{knife_V5478G}`.

## Solution

If we search for `Jonathan Abigdail` on Instagram, we get a few results. One of the top results is of a user called `Dr. Jonathan Abigdail III`, which matches the clues from the poem: `At doctor's Jonathan Abigdail the third they wept`.

![jonathan_abigdail](/images/writeups/idekCTF2022/osint_crime_confusion/jonathan_abigdail.png)

His only post contains an image of an eye with a hashtag in the description.

![insta_1](/images/writeups/idekCTF2022/osint_crime_confusion/insta_1.png)

Clicking the hashtag brings us to another image. The image is another eye but neither the image itself nor the description contains anything of interest. However, we see that the image is posted by a different user with the username `hjthepainteng`.

![insta_2](/images/writeups/idekCTF2022/osint_crime_confusion/insta_2.png)

Clicking the username brings us to the user's profile. The user has another post of an image of a water ripple.

![insta_3](/images/writeups/idekCTF2022/osint_crime_confusion/insta_3.png)

Checking out the image, we find a number of clues from the description, namely the `great_paintball_portugal` competition, and something being listed for sale on eBay.

![insta_4](/images/writeups/idekCTF2022/osint_crime_confusion/insta_4.png)

I initially thought that the `great_paintball_portugal` competition was a real location or event, and tried to google keywords related to this, to no success. Next I tried searching on [what3words](https://what3words.com/) as I hypothesised that it could be a 3 word address.

Finally, I tried to see if `great_paintball_portugal` could be a username on eBay. To that effect, eBay's user profiles are accessible at `https://www.ebay.com/usr/<username>`. Replacing `<username>` with `great_paintball_portugal` brings us to the user's profile.

As we can see in the can see in the description, there is a link to a Wix website.

![ebay](/images/writeups/idekCTF2022/osint_crime_confusion/ebay.png)

Navigating to the website, we see only see a home page with no sign of the blog post as mentioned in the eBay description.

![wix_1](/images/writeups/idekCTF2022/osint_crime_confusion/wix_1.png)

After a while, I tried append `/blog` to the URL, and it worked! We can see a single post on the website.

![wix_2](/images/writeups/idekCTF2022/osint_crime_confusion/wix_2.png)

Lastly, the flag for this challenge can be found by opening the blog post to see its full content.

![wix_3](/images/writeups/idekCTF2022/osint_crime_confusion/wix_3.png)

Flag: `idek{TGPP_WCIYD}`

You can read the other parts of this challenge [here](https://jacksun.dev/tags/osint-crime-confusion/)