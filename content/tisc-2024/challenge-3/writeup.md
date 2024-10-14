---
title: Challenge 3 - Digging Up History
draft: true
tags:
  - tisc24
  - tisc24-challenge3
  - digging_up_history
---
# Writeup

![[Pasted image 20240930053657.png]]

![[Pasted image 20240930053707.png]]

![[Pasted image 20240930053712.png]]

![[Pasted image 20240930053717.png]]

![[Pasted image 20240930053726.png]]
![[Pasted image 20240930053743.png]]

1. **Steps to solve the challenge**
    
    - mounted the AD1 disk image using FTK Imager (via Windows VM). dumped the logical file contents into a folder and transferred it to my Linux system (to use CLI).
    - under the Documents and Settings\csitfan1\Application Data folder, contains a lot of web browser data, with Mypal68 being the biggest in size.
    - Did a recursive grep search to dump all the urls visited in that browser and put it in a text file:
    
    `strings Local Settings/Application Data/Mypal68/Profiles/a80ofn6a.default-default/cache2/* | grep -ar -Eo "(http|https)://[a-zA-Z0-9./?=_%:-]*" | cut -d ":" -f 2,3 | sort -u > ~/Documents/tisc_2024/challenge3/urlhistory_mypal68.txt`
    
    - went through the URL, saw this URL that's most likely the flag: [https://csitfan-chall.s3.amazonaws.com/flag.sus](https://csitfan-chall.s3.amazonaws.com/flag.sus "https://csitfan-chall.s3.amazonaws.com/flag.sus")
    - Downloaded it and it turned out to be a base64 string. Decoded the content with base64 -d:
    
    `base64 -d flag.sus`
    
    - Tada, flag ![[Pasted image 20240930053814.png]]

    
