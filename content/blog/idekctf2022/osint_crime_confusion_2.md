---
title: "idekCTF 2022: Osint Crime Confusion 1-4"
date: 2023-01-16T17:37:37+08:00
draft: true
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

## Description

- Name: Osint Crime Confusion 2: W as in Weapon
- Category: OSINT
- 476 pts, 50 solves
- Written by `Franfrancisco9#0105`

Now that you found where, can you help me find what was the weapon of the crime? It has something to do with a university of science.

Note: Previous links or accounts might be useful.

## Solution

The description of this challenge gives us a hint that the weapon is related to a university of science. Remember our boy Heather James from the previous challenge? His Instagram bio mentions the `University of Dutch ThE of Topics in Science (UThE_TS)`.

![heather_james](/images/writeups/idekCTF2022/osint_crime_confusion/heather_james.png)

I managed to find a profile for this university on Twitter using the handle `@UThE_TS`.

![twitter_1](/images/writeups/idekCTF2022/osint_crime_confusion/twitter_1.png)
![twitter_2](/images/writeups/idekCTF2022/osint_crime_confusion/twitter_2.png)
![twitter_3](/images/writeups/idekCTF2022/osint_crime_confusion/twitter_3.png)

![google_1](/images/writeups/idekCTF2022/osint_crime_confusion/google_1.png)
![google_2](/images/writeups/idekCTF2022/osint_crime_confusion/google_2.png)
![google_3](/images/writeups/idekCTF2022/osint_crime_confusion/google_3.png)
![google_4](/images/writeups/idekCTF2022/osint_crime_confusion/google_4.png)
![google_5](/images/writeups/idekCTF2022/osint_crime_confusion/google_5.png)

`https://pads.ccc.de/ep/pad/view/ro.lvGC01KAJWI/rev.354`
![chaos_1](/images/writeups/idekCTF2022/osint_crime_confusion/chaos_1.png)
![chaos_2](/images/writeups/idekCTF2022/osint_crime_confusion/chaos_2.png)
![chaos_3](/images/writeups/idekCTF2022/osint_crime_confusion/chaos_3.png)

Flag: `idek{HSTM_X!#$}`