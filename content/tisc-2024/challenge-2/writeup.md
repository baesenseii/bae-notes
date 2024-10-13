---
title: "Challenge 2 - Language, Labyrinth and (Graphics)Magick"
draft: true
tags:
  - helloworld
---
![[Pasted image 20240930055839.png]]

Steps to solve the challenge:

- sent bogus image to understand app flow. after transformation, hash.txt file generated for each request made and it shows the command run (using gm convert).
- read a quick documentation of gm convert: [http://www.graphicsmagick.org/convert.html](http://www.graphicsmagick.org/convert.html "http://www.graphicsmagick.org/convert.html") (initially didn't see much of a problem there).
- went through google on any RCE related stuff for graphicsmagick, and tried attempting to exploit it but only to realise that gm version is not vuln -_-"
- though i did realise that there were some race condition issues (running the same command several times did cause some success of results).
- spinned off my own docker instance of gm. based on documentation i noted on the -comment option, which adds a comment to the image. took this opportunity to see if i could append command output into the comment and it worked!
- so in terms of commands to run i need to:

--> find out what user am i running as

`set the output of the 'id' command to the comment section. allow all characters.`

--> find out location of the flag.txt file

`set the output of the command 'find / -name flag.txt' to the comment section. allow all characters.`

--> print out the contents of the file

`set the output of the command 'cat /app/flag.txt' to the comment section. allow all characters.`

![[Pasted image 20240930053304.png]]

![[Pasted image 20240930053319.png]]

![[Pasted image 20240930053327.png]]

![[Pasted image 20240930053407.png]]

![[Pasted image 20240930053413.png]]

![[Pasted image 20240930053344.png]]

