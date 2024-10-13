---
title: "How to publish Obsidian notes with Quartz on GitHub Pages"
draft: true
tags:
  - 
---
![[Pasted image 20240930055121.png]]

***Card structure (based on 135 bytes)***
byte 0 to byte 4 => signature "AGPAY"
byte 5 to byte 6 => version
byte 7 to byte 38 => encryption key
byte 39 to byte 48 => reserved
byte 49 to byte 64 => iv
byte 65 to byte 112 => encrypted data
byte 113 to byte 117 => footer signature "ENDAGP"
byte 118 to 134 => checksum (iv + encrypted data => md5hash) (edited)

ベイ先生 — 15/09/2024 21:32
encrypted data (sample size is 48 bytes) contains cleartext data which make up the following:
byte 0 to byte 15 => cardNumber
byte 16 to byte 19 => some standard byte data (no idea what it is)
byte 20 to byte 23 => card expiry date (unsigned 32-bit int)
byte 24 to byte 31 => balance of card (unsigned 32-bit int, big indian notation)

```
#!/usr/bin/python3

from Crypto.Cipher import AES
import hashlib
import base64
import struct

def constructCCDetails(cardNumber, standardT, cardExpiryDate, balance):
	add_padding = '\x10' * 16
	ccDetails = bytearray(cardNumber) + bytearray(standardT) + bytearray(cardExpiryDate) + bytearray(balance) + bytearray(add_padding.encode())
	return bytes(ccDetails)

def decrypt(data,key,iv):
	cipher = AES.new(key, AES.MODE_CBC, iv)
	decryptedtext = cipher.decrypt(data)   # Base64 decode the ciphertext
	return decryptedtext

def encrypt(data,key,iv):
	cipher = AES.new(key, AES.MODE_CBC, iv)
	encryptedtext = cipher.encrypt(data)
	return encryptedtext

def calculate_checksum(data):
    md5_hash = hashlib.md5(data)
    return md5_hash.digest()

###
# 
# Objective is just to replace the balance value and reconstruct the credit card details properly.
#
###

sample = open("./testcard.agpay","rb")

# get total bytes length
byte_length_sample = len(sample.read())

### keeping the items
# get signature
sample.seek(0)
signature = sample.read(5)

# get version
sample.seek(5)
version = sample.read(2)

# get encryptionKey
sample.seek(7)
key = sample.read(39-7)

# get reserved
sample.seek(39)
reserved = sample.read(49-39)

# get footer signature
sample.seek(byte_length_sample - 22)
footerSignature = sample.read(6)

# get checksum
sample.seek(byte_length_sample - 16)
checksum = sample.read(16)

# get iv
sample.seek(49)
iv = sample.read(65-49)

# get encrypted data
sample.seek(65)
encryptedData = sample.read(byte_length_sample-22-65)

sample.seek(113)

decryptedData = decrypt(encryptedData,key,iv)

cardNumber = decryptedData[0:16]
standardT = decryptedData[16:20]
cardExpiryDate = decryptedData[20:24]
balance = decryptedData[24:32]

# Define the offset and endianness (little-endian by default) => this is to handle the balance value which is in big endian notation.
offset = 0 
little_endian = False

v_balance = struct.unpack('>Q' if not little_endian else '<Q',balance)[0]
new_balance = struct.pack('>Q' if not little_endian else '<Q',313371337)
v_new_balance = struct.unpack('>Q' if not little_endian else '<Q',new_balance)[0]

##################################################################
#
# construct new credit card details
#
##################################################################
new_cc_details = constructCCDetails(cardNumber,standardT,cardExpiryDate,new_balance)
new_cc_details_enc = encrypt(new_cc_details, key, iv)
new_checksum = calculate_checksum(bytes(bytearray(iv) + bytearray(new_cc_details_enc)))

new_cc = bytearray(signature) + bytearray(version) + bytearray(key) + bytearray(reserved) + bytearray(iv) + bytearray(new_cc_details_enc) + bytearray(footerSignature) + bytearray(new_checksum)

f2 = open('newcard.agpay','wb')
f2.write(bytes(new_cc))
f2.close()
sample.close()

```

![[Pasted image 20240930054003.png]]
![[Pasted image 20240930054042.png]]
## Steps taken:
- downloaded the sample agpay card file.
- created a python script that replicates the extraction of bytes for the credit card fields, and then regenerates a new one based on the new balance value.
- upload the new credit card file into the app, voila, flag shown.