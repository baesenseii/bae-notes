```
---
title: "Challenge 1 - Navigating the Digital Labyrinth"
draft: false
tags:
  - tisc24
  - tisc24-challenge1
  - navigating_the_digital_labyrinth
---
```

Finally having the time to do a write-up for the TISC 2024 challenges, starting with the first challenge called "Navigating the Digital Labyrinth":

![[challenge-1-1.png]]

Based on the description provided, there is an online persona that we need to search over various social media platforms, and it turns out this is the only one that matches the user 'vi_vox223':

![[Pasted image 20240930050231.png]]

Just like any other typical CTF challenge that involves social media, I'm sure that there's more platforms that are in use. As we can see in the picture above, there is some Instagram story labelled 'Discord', which showed two relevant pictures:

![[Pasted image 20240930050306.png]]![[Pasted image 20240930050312.png]]

Based on the photos above we can deduce the following:
- There is a presence of a Discord bot that we need to communicate with (which means that we will need a Discord server in this case)
- To interact with the Discord bot, users of the Discord server will have to be granted with the D0PP3L64N63R role.

Setting a Discord server is pretty easy using the Discord UI, but in order to add a Discord bot to the server, we'll first need to craft out the Discord Bot invite link (luckily for us we can craft out as the Bot ID was provided to us, along with the following resource reference below):
- https://discordjs.guide/preparations/adding-your-bot-to-servers.html#bot-invite-links

![[Pasted image 20240930052328.png]]

In addition, we need to create the D0PP3L64N63R role within the Discord server to allow us to interact with the Discord Bot (maybe we might not need it perhaps, but just create it for now):

![[Pasted image 20240930050527.png]]

After adding the Discord Bot to the Discord server, send the following message to any text channel within the Discord server:
```
!help
```

![[Pasted image 20240930050536.png]]

At first it was only two commands, however after assigning the user with the D0PP3L64N63R role and typing the command once again:

![[Pasted image 20240930050545.png]]

Especially for people like me, the ```list``` command looks very enticing:

![[Pasted image 20240930050552.png]]

After going through the files, the Update_030624.eml file was that one document that deemed to be the 'most interesting one':

![[Pasted image 20240930050838.png]]

Opening the .eml file using Outlook and the following email message was displayed:

![[Pasted image 20240930050904.png]]

There are two parts to this email that requires further investigation:
- The 3 IDs provided (based on the email it has to do something with 'Uber's cutting-edge geospatial technology')
- There is a 'secure communication channel' located in https://www.linkedin.com/company/the-book-lighthouse

For the 3 IDs provided, those are apparently map coordinates using the H3 global grid system (https://h3geo.org/), developed by Uber for optimization purposes of providing Uber services more effectively and efficiently for their users. Each ID corresponds to the following:
- Resolution
- Latitude
- Longitude
- Base Cell
- etc.

![[Pasted image 20240930052903.png]]

Since we are mainly interested on map points, we are more interested in the Latitude and Longitude of the 3 IDs provided:
- 8c1e806a3ca19ff => 41.544426767, 12.994242633
- 8c1e806a3c125ff => 41.544551463, 12.994414040 
- 8c1e806a3ca1bff => 41.544383254, 12.994469395

Calculating the midpoint of the 3 map coordinates above just involves two calculations:
- Midpoint Latitude = Average of the 3 Latitude Values = 41.544453828
- Midpoint Longitude = Average of the 3 Longitude Values = 12.994375356

Entering the midpoint map coordinates in Google Maps brings us to some place in Italy:

![[Pasted image 20240930050958.png]]

Now as for discovering the 'secret communication channel', looking through the LinkedIn pages brings us to a Telegram Bot called 'TBL_DictioNaryBot', in which we can easily communicate with using a valid Telegram account (used my personal one):

![[Pasted image 20240930051327.png]]

After multiple tries, it turns out that the answer to send to the Telegram bot was the location name 'Quercia secolare', as the email did state that the person was close to the midpoint -_-"

![[Pasted image 20240930050950.png]]

And VOILA! The flag presents itself: TISC{OS1N7_Cyb3r_InV35t1g4t0r_uAhf3n}