---
title: "De1CTF 2019 - Web - ssrf_me"
date: 2019-08-05
draft: false
---

# De1CTF 2019 – ssrf_me

**Category :** Web
**Description :** SSRF ME TO GET FLAG.
http://139.180.128.86/
**Hint :** hint for [SSRF Me]: flag is in ./flag.txt


## Write-up
As suggested by title and description, we had to make a ssrf attack to get the flag.
The home page of the website returned the server's source code:
```python
#! /usr/bin/env python
#encoding=utf-8
from flask import Flask
from flask import request
import socket
import hashlib
import urllib
import sys
import os
import json
reload(sys)
sys.setdefaultencoding('latin1')

app = Flask(__name__)

secert_key = os.urandom(16)


class Task:
    def __init__(self, action, param, sign, ip):
        self.action = action
        self.param = param
        self.sign = sign
        self.sandbox = md5(ip)
        if(not os.path.exists(self.sandbox)):          #SandBox For Remote_Addr
            os.mkdir(self.sandbox)

    def Exec(self):
        result = {}
        result['code'] = 500
        if (self.checkSign()):
            if "scan" in self.action:
                tmpfile = open("./%s/result.txt" % self.sandbox, 'w')
                resp = scan(self.param)
                if (resp == "Connection Timeout"):
                    result['data'] = resp
                else:
                    print resp
                    tmpfile.write(resp)
                    tmpfile.close()
                result['code'] = 200
            if "read" in self.action:
                f = open("./%s/result.txt" % self.sandbox, 'r')
                result['code'] = 200
                result['data'] = f.read()
            if result['code'] == 500:
                result['data'] = "Action Error"
        else:
            result['code'] = 500
            result['msg'] = "Sign Error"
        return result

    def checkSign(self):
        if (getSign(self.action, self.param) == self.sign):
            return True
        else:
            return False


#generate Sign For Action Scan.
@app.route("/geneSign", methods=['GET', 'POST'])
def geneSign():
    param = urllib.unquote(request.args.get("param", ""))
    action = "scan"
    return getSign(action, param)


@app.route('/De1ta',methods=['GET','POST'])
def challenge():
    action = urllib.unquote(request.cookies.get("action"))
    param = urllib.unquote(request.args.get("param", ""))
    sign = urllib.unquote(request.cookies.get("sign"))
    ip = request.remote_addr
    if(waf(param)):
        return "No Hacker!!!!"
    task = Task(action, param, sign, ip)
    return json.dumps(task.Exec())
@app.route('/')
def index():
    return open("code.txt","r").read()


def scan(param):
    socket.setdefaulttimeout(1)
    try:
        return urllib.urlopen(param).read()[:50]
    except:
        return "Connection Timeout"



def getSign(action, param):
    return hashlib.md5(secert_key + param + action).hexdigest()


def md5(content):
    return hashlib.md5(content).hexdigest()


def waf(param):
    check=param.strip().lower()
    if check.startswith("gopher") or check.startswith("file"):
        return True
    else:
        return False


if __name__ == '__main__':
    app.debug = False
    app.run(host='0.0.0.0',port=80)
```

I analysed the code starting by the entry points (REST Services). We could do two actions via this services: 

1. Sign a request calling /geneSign
2. Execute an action calling /De1ta



### /geneSign analysis:

This part of the source code was pretty simple. We could see that: 
1. We could only sign a 'scan' action 
2. There was no filtering on the 'param' field therefore we could pass anything 
3. Our input was directly used to build the request signature

### /De1ta analysis: 

This part of the code contained the vulnerability. We could see that: 
1. We could perform two different actions on the server "scan" and "read" 
2. "scan" allowed us to write data in a temporary file via scan() method.
3. scan() execute  ```urllib.urlopen(param).read()[:50]``` which contained 'param' field (we have control on it)
4. To execute a command, we had to provide its signature (compared to one computed server-side) 
5. Actions were executed on a temp file named ./result.txt 

From here we could deduce the attack: Write the content of ./flag.txt in result.txt (scan action) and then read result.txt to get the flag (read action). It looked easy but there were two problems:

- There was a waf() function which prevented us to use ```gopher or url starting by file:``` 
- We needed to sign our 'read' action but /geneSign only worked for 'scan' 

### Exploit 

#### Write the content of ./flag.txt

 According to ```CVE-2019-9948``` (https://www.cvedetails.com/cve/CVE-2019-9948/) we could use local_file: scheme to bypass the waf()function.In fact the function prevented only 'file:' scheme
 Therefore it was pretty easy to write the flag in result.txt.
1. Get the signature for our request: GET  /geneSign?param=local_file:flag.txt
2. Call url to write the flag: /De1ta?param=local_file:flag.txt with follwing cookies action=scan and signature={result_of_step_1}


#### Read the content of ./result.txt

I struggled for a while to find a way to sign a 'read' action. In my mind we could only execute one action by request: either 'read' or 'scan' . But I finally noticed that we could execute both in a single request. In fact: 
- exec() contained neither a 'switch... case..' nor an 'if... else if .. ' but two simple 'if' one after the other
- It checked if 'action' contained 'scan' (respectively contains 'read') => action could be something like 'scanread' 


From /geneSign analysis we found that the signature was based on a parameter we could control: 
- md5 signature was build from the following: secert_key + param + action
As the action was forced to 'scan' we could bypass this restriction by putting 'read' at the end of 'param field'

=> secert_key + 'hackread' + 'scan'
=> secert_key + hackreadscan'

From here signing :
    action=scan
    param=hackread
or: 
    action=readscan
    param=hack

provided the exact same hash signature. We could sign a read request this way. To bypass the signature protection we just had to ask /geneSign to sign the first one and call /De1ta using the second one. 

1. Let's sign our request: GET  /geneSign?param=hackread
    => I put 'hack' here just to have something in 'param' field later. But I suppose that it would have worked without it 
2. Call url to read the flag: /De1ta?param=hack with following cookies action=readscan and signature={result_of_step_1}
    => Notice that 'read' is no more at the end of 'param' field but at the start of the action 


=> The flag was in the REST call response: {"code": 200, "data": "de1ctf{27782fcffbb7d00309a93bc49b74ca26}"}

