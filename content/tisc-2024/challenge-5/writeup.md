```
---
title: "How to publish Obsidian notes with Quartz on GitHub Pages"
draft: true
tags:
  - 
---
```

![[Pasted image 20240930055143.png]]

HUGE CREDIT TO MAH BUDDY THAT I MANAGED TO PULL INTO THIS HELLHOLE cExplr for this TISC challenge (I seriously wouldn't have be able to do this without him man, and I learnt the MOST out of it). Solving the challenge involves 3 parts:

- reverse engineering the flash dump
- reverse engineering i2c communication with the implant

**_rev dump_**

- challenge starts off by providing a dump file of the i2c implant (command syntax used for dumping the image was via esptool, which meant that the dump we're dealing with had something to do with ESP32)
- tried and tested out a number of tools, and the one that had the most success of carving out the various partitions is this tool: [https://github.com/BlackVS/esp32knife](https://github.com/BlackVS/esp32knife "https://github.com/BlackVS/esp32knife")

An interesting output was an ELF file, which was loaded onto mah fav disassembler Ghidra, and that's when I knew, idk wtf I was doing.

- But took me a while to find out what I was reading using the various strings from the .romdata segment (there was a fake flag and some other strings)
- but yeah advised further that there's no point analyzing more if we can't figure out how to communicate with the implant first.

**Rev i2c comms**

- connected to chals.tisc24.ctf.sg via 61622 and saw a very nice ASCII diagram that indicates I2C.
- there are 3 commands: SEND, RECV, EXIT.
- after painstakingly reading so about i2c comms, there are two things that we need to find out: what addressing scheme is it using, and what is the slave address of the i2c implant.
- thanks to the example written for the SEND command, we can establish that it is using a 7-bit addressing scheme (MSB contains 7-bit address value + 1-bit to determine write (0) or read (1) operation)
- at first I simply tried to just send bogus data and received it immediately, and again hit with a damn rock wall.
- till yesterday cExplr told me he had an amazing breakthrough whereby 3 commands were sent (after numerous automated trial and errors):
```
SEND D2 46 
SEND D3 
RECV 20
```

- returned bytes that are NOT ZERO. And it did make sense on the order of the commands given. Just a breakdown of what was sent:

-- payload 46 was written to address 0x69 (SEND D2 46) on the I2C bus -- a read condition was also sent to the I2C bus so that we can read off the bytes from the same I2C bus via our buffer -- the RECV command reads the bytes off the buffer (which in this case prints out some weird bytes that are finally NOT ZERO)

- and it turns out that we've accidentally found the slave address too, hurray for that! Now we gotta try to make sense of what we have between the disassembled code and the behavior we observed.


**back to the app**

- Now that we know how to talk to the app, it's time to figure out what caused the output.
- After going through the disassembled ESP32 code, a particular function of interest was discovered @ FUN_400d1614 that was spitting output.
- realised that using the 3 commands above, sending the payload 4D results in the CrapTPM banner to be displayed, whereas payload 46 results in an "encrypted" form of the flag.
- the encryption involves a byte-level XOR encryption as seen in the disassembled code, or what cExplr has deduced it to be a rolling XOR encryption (as seen in the function).
- what makes it even more interesting is that the generation of each key uses a random seed that is an unsigned short variable (16-bit), but when the XOR encryption took place, the resulting value was a byte worth (8-bit) ==> NOT the entire KEY was used for the XOR operation, only 1 byte worth.
- As we know that the first 5 characters of the flag start with 'TISC{', we could do a XOR operation with the encrypted value for each character, however due to the bit shifts on an unsigned short variable during the key generation, reversing it back will not be possible due to possible data loss.
- so the approach here is this (not sure how to explain this):

--> the first byte of the XORed result between the encrypted value and the word "T", do a brute force attack of the 2nd byte (most significant byte) till the key results to the next byte of the XORed result between the encrypted value and the letter "I". --> once a key is discovered for the above, function gets looped again with the new key to see if the next result would be the XORed result between the 2nd encrypted byte value and the word "I". --> we have samples up to the first 5 characters, so if we find the correct key, it should work.

- once the full key is found (for each byte), do a XOR encryption between the discovered full key and the encrypted value from the TPM = BAM ZE FLAG
![[Pasted image 20240930054335.png]]

```
# key is global and will be reused

def custom_rolling():
    global key
    temp = (key << 7 ^ key) & 0xffff
    temp =(temp >> 9 ^ temp) & 0xffff  
    key = (temp << 8 ^ temp) & 0xffff

    return key

# for i in range(0xff):
#     check = []
#     key = 0x4f
#     key = (i<<8)|key
#     temp_key = key

#     custom_rolling()
#     if (key&0xff) == 0xaa:
#         print("key : ", hex(temp_key))
#         print("FOUND")

#after 1 round of rolling
key = 0x6d4f

possible_key = [0x6d4f]
print("key : ", hex(key))

custom_rolling()
print("\nNew key : ", hex(key))
possible_key.append(key)

custom_rolling()
print("\nNew key : ", hex(key))
possible_key.append(key)

custom_rolling()
print("\nNew key : ", hex(key))
possible_key.append(key)

custom_rolling()
print("\nNew key : ", hex(key))
possible_key.append(key)

custom_rolling()
print("\nNew key : ", hex(key))
possible_key.append(key)

custom_rolling()
print("\nNew key : ", hex(key))
possible_key.append(key)

custom_rolling()
print("\nNew key : ", hex(key))
possible_key.append(key)

custom_rolling()
print("\nNew key : ", hex(key))
possible_key.append(key)

custom_rolling()
print("\nNew key : ", hex(key))
possible_key.append(key)

custom_rolling()
print("\nNew key : ", hex(key))
possible_key.append(key)

custom_rolling()
print("\nNew key : ", hex(key))
possible_key.append(key)

custom_rolling()
print("\nNew key : ", hex(key))
possible_key.append(key)

custom_rolling()
print("\nNew key : ", hex(key))
possible_key.append(key)

custom_rolling()
print("\nNew key : ", hex(key))
possible_key.append(key)

custom_rolling()
print("\nNew key : ", hex(key))
possible_key.append(key)

print(len(possible_key))

TPM_val = [0x1b,0xe3, 0xe3, 0xdd, 0xe3, 0x9c, 0xa8, 0x0a, 0x07, 0x27, 0xad, 0xf4, 0xc1, 0x0f, 0x31, 0xe8]

flag = ""
for i in range(len(possible_key)):
    flag += chr(TPM_val[i] ^ (possible_key[i] & 0xff))

print(flag)

```