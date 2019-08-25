---
title: "RedpwnCTF - Misc - Tux Trivia Show"
date: 2019-08-16
draft: false
---

RedpwnCTF 2019 – Tux Trivia Show

**Category :** Misc
**Description :** Win Tux Trivia Show!

nc chall2.2019.redpwn.net 6001

## Write-up

This challenge was not difficult but really annoying due to restrictive spelling.

It consists in a really large amount of question about  capitals (country/usa's states) 

We create a simple text file containing the required data as follow: 

```
Switzerland:Bern
Germany:Berlin
France:Paris
... ... .. 
```

and then we made a script to answer the questions automatically: 

```python
#!/usr/bin/python3

from pwn import *
import time

r = remote("chall.2019.redpwn.net", 6001)

while True:
    try:
        request=r.recvline(timeout=100).decode("utf-8")  
        print(request)  
        if "What is the capital of" in request:
            search= request[23:][:-2] +":"
            with open("capital.txt") as datas:
                for line in datas:
                    if search in line:
                        if str(line[0]) == str(search[0]) and str(line[1]) == str(search[1]):
                            answer = line.split(":")[1]
                            print('answer: ',answer)
                            r.send(answer)
    except:
        time.sleep(10)
        request = r.recvall().decode("utf-8")
        print(request)
        exit()
```


flag: flag{TUX_tr1v1A_sh0w+m3st3r3d_:D}


