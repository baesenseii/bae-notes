---
title: Challenge 4 - AlligatorPay
draft: false
tags:
  - tisc24
  - tisc24-challenge4
  - alligatorpay
---
# Writeup

![[Pasted image 20240930055121.png]]
I guess the one thing that i will remember from this challenge is this:

"
Alligator Pay, Alligator Pay
Swipe me up, you're on your way.
Safe and fast, every day,
Alligator leads the way!
"
HAHAHAHAHA! Anyway back to the writeup.

So we are presented with a web application that takes in a specific Alligator Pay Card and present the current balance of the card. The sample testcard.agpay file is as follows:

![[Pasted image 20240930054003.png]]

As there were no HTTP requests being sent upon uploading the testcard.agpay file into the web application, it is highly possible that the whole web application's logic was mainly client-side. And as it turns out...

```
const arrayBuffer = await file.arrayBuffer();
        const dataView = new DataView(arrayBuffer);

        const signature = getString(dataView, 0, 5);
        if (signature !== "AGPAY") {
          alert("Invalid Card");
          return;
        }
        const version = getString(dataView, 5, 2);
        const encryptionKey = new Uint8Array(arrayBuffer.slice(7, 39));
        const reserved = new Uint8Array(arrayBuffer.slice(39, 49));

        const footerSignature = getString(
          dataView,
          arrayBuffer.byteLength - 22,
          6
        );
        if (footerSignature !== "ENDAGP") {
          alert("Invalid Card");
          return;
        }
        const checksum = new Uint8Array(
          arrayBuffer.slice(arrayBuffer.byteLength - 16, arrayBuffer.byteLength)
        );

        const iv = new Uint8Array(arrayBuffer.slice(49, 65));
        const encryptedData = new Uint8Array(
          arrayBuffer.slice(65, arrayBuffer.byteLength - 22)
        );

        const calculatedChecksum = hexToBytes(
          SparkMD5.ArrayBuffer.hash(new Uint8Array([...iv, ...encryptedData]))
        );

        if (!arrayEquals(calculatedChecksum, checksum)) {
          alert("Invalid Card");
          return;
        }

        const decryptedData = await decryptData(
          encryptedData,
          encryptionKey,
          iv
        );

        const cardNumber = getString(decryptedData, 0, 16);
        const cardExpiryDate = decryptedData.getUint32(20, false);
        const balance = decryptedData.getBigUint64(24, false);
[21:42]
ocument.getElementById("cardNumber").textContent =
          formatCardNumber(cardNumber);
        document.getElementById("cardExpiryDate").textContent =
          "VALID THRU " + formatDate(new Date(cardExpiryDate * 1000));
        document.getElementById("balance").textContent =
          "$" + balance.toString();
        console.log(balance);
        if (balance == 313371337) {
          function arrayBufferToBase64(buffer) {
            let binary = "";
            const bytes = new Uint8Array(buffer);
            const len = bytes.byteLength;
            for (let i = 0; i < len; i++) {
              binary += String.fromCharCode(bytes[i]);
            }
            return window.btoa(binary);
          }
```

After carefully studying this god-foresaken piece of JavaScript code, it seems that to get the flag, we need to get the balance of the card to be 313371337. Setting the balance of the AlligatorPay card isn't that straightforward because of the various integrity-level checks + encryption that is being done during the construction of the card. 

As such, a detailed breakdown of the card's structure has been documented by yours truly:

***Card structure (based on 135 bytes)***
- byte 0 to byte 4 => signature "AGPAY"
- byte 5 to byte 6 => version
- byte 7 to byte 38 => encryption key
- byte 39 to byte 48 => reserved
- byte 49 to byte 64 => iv
- byte 65 to byte 112 => encrypted data
- byte 113 to byte 117 => footer signature "ENDAGP"
- byte 118 to 134 => checksum (iv + encrypted data => md5hash) (edited)

The decrypted data (48 bytes in its encrypted state) contains the following data structure:
- byte 0 to byte 15 => cardNumber
- byte 16 to byte 19 => some standard byte data (no idea what it is)
- byte 20 to byte 23 => card expiry date (unsigned 32-bit int)
- byte 24 to byte 31 => balance of card (unsigned 32-bit int, big indian notation)
- byte 32 to byte 48 => i don't really care.

Based on this understanding, and my scripting brain, the following Python script was used to reconstruct a new AlligatorPay card with the intended balance of 313371337 (this requires the testcard.agpay as a reference card template):

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

The new card was then uploaded into the web application, and there you go, le flag:

![[Pasted image 20240930054042.png]]

Flag: TISC{533_Y4_L4T3R_4LL1G4T0R_a8515a1f7004dbf7d5f704b7305cdc5d}

# Thoughts

I would say this is a really good brain teaser to work on one of the most fundamental skills of reverse engineering, which is to build your ACTUAL understanding of the application on how it works (from the submission of the AG card to the displaying of balance on the site).

I also took this opportunity to practise my scripting skills (yes yes, i know people keep talking about using GenAI to create the script, but i believe that by doing so, you shortchange the ability to think logically in terms of the flow of data when building your script). I know that many people would rather just edit the provided AG card using hex editors and such, but the idea of scripting out the construct of the new AG card personally helps you as a security researcher to logically provide a solution to anyone without effort required.

I mean, even taking the effort to do a writeup something like this takes a considerable amount of effort for me because it requires me to think about the flow / thought process when approaching the challenge and how do i convey it right so that people can comprehend the writeup after reading it.