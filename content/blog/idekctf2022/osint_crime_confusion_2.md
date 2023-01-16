---
title: "idekCTF 2022: Osint Crime Confusion 2: W as in Weapon"
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

- Name: Osint Crime Confusion 2: W as in Weapon
- Category: OSINT
- 476 pts, 50 solves
- Written by `Franfrancisco9#0105`

Now that you found where, can you help me find what was the weapon of the crime? It has something to do with a university of science.

Note: Previous links or accounts might be useful.

## Solution

The description of this challenge gives us a hint that the weapon is related to a university of science. Remember our boy Heather James from the previous challenge? His Instagram bio mentions the `University of Dutch ThE of Topics in Science (UThE_TS)`.

![heather_james](/images/writeups/idekCTF2022/osint_crime_confusion/heather_james.png)

I managed to find a profile for this university on Twitter using the handle `@UThE_TS`. The university has a few tweets, however, none seems particularly interesting.

![twitter_1](/images/writeups/idekCTF2022/osint_crime_confusion/twitter_1.png)

Clicking the 3 dots on the profile reveals a number of options. One of which is to view lists.

![twitter_2](/images/writeups/idekCTF2022/osint_crime_confusion/twitter_2.png)

There is a single list called `The List Test`. Clicking on the list reveals a description that contains some interesting clues.

![twitter_3](/images/writeups/idekCTF2022/osint_crime_confusion/twitter_3.png)

I first tried to search about German Chaos Pad on Google. However, I didn't find anything useful.

I then tried googling `/ep/pad/view/`. This led me to a website called TitanPad.

![google_1](/images/writeups/idekCTF2022/osint_crime_confusion/google_1.png)

While the website has unfortunately shut up, it didn't leave without leaving us the next clue: EtherPad.

![google_2](/images/writeups/idekCTF2022/osint_crime_confusion/google_2.png)

Checking out the EtherPad website, I find that it is an open source online editor project that can be hosted by anyone.

![google_3](/images/writeups/idekCTF2022/osint_crime_confusion/google_3.png)

Thus, logically the next step is to find the EtherPad instance that was used by the university. We have the clues to look in the `german chaos pad`. Googling `German Chaos Etherpad`, we find the following website, with an link to the `Chaos Computer Club` pad.

![google_4](/images/writeups/idekCTF2022/osint_crime_confusion/google_4.png)

![google_5](/images/writeups/idekCTF2022/osint_crime_confusion/google_5.png)

I Then tried appending the pad link to the domain name as follows `https://pads.ccc.de/ep/pad/view/ro.lvGC01KAJWI/rev.354` and hit enter. Annnd bingo!

![chaos_1](/images/writeups/idekCTF2022/osint_crime_confusion/chaos_1.png)

The bar at the top is a timeline/history scrubber. Moving the scrubber to the very right reveals the flag format: `4 initials of the object plus  "_X!#$" is the key ( idek{4CapitalLetters_X!#$}`.

![chaos_2](/images/writeups/idekCTF2022/osint_crime_confusion/chaos_2.png)

Moving the scrubber to version 338 reveals the murder weapon, the `HUBBLE SPACE TELESCOPE MODEL`.

![chaos_3](/images/writeups/idekCTF2022/osint_crime_confusion/chaos_3.png)

Combining the first letter of the murder weapon gives us the final flag.

Flag: `idek{HSTM_X!#$}`

You can read the other parts of this challenge [here](https://jacksun.dev/tags/osint-crime-confusion/)