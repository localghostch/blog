---
title: "De1CTF 2019 - Crypto - xorz"
date: 2019-08-05
draft: false
---

# De1CTF 2019 – xorz

**Category :** Crypto
**Description :** xorzzz...

## Write-up

For this challenge, we got the following python script :
```python
from itertools import *
from data import flag,plain

key=flag.strip("de1ctf{").strip("}")
assert(len(key)<38)
salt="WeAreDe1taTeam"
ki=cycle(key)
si=cycle(salt)
cipher = ''.join([hex(ord(p) ^ ord(next(ki)) ^ ord(next(si)))[2:].zfill(2) for p in plain])
print cipher
# output:
# 49380d773440222d1b421b3060380c3f403c3844791b202651306721135b6229294a3c3222357e766b2f15561b35305e3c3b670e49382c295c6c170553577d3a2b791470406318315d753f03637f2b614a4f2e1c4f21027e227a4122757b446037786a7b0e37635024246d60136f7802543e4d36265c3e035a725c6322700d626b345d1d6464283a016f35714d434124281b607d315f66212d671428026a4f4f79657e34153f3467097e4e135f187a21767f02125b375563517a3742597b6c394e78742c4a725069606576777c314429264f6e330d7530453f22537f5e3034560d22146831456b1b72725f30676d0d5c71617d48753e26667e2f7a334c731c22630a242c7140457a42324629064441036c7e646208630e745531436b7c51743a36674c4f352a5575407b767a5c747176016c0676386e403a2b42356a727a04662b4446375f36265f3f124b724c6e346544706277641025063420016629225b43432428036f29341a2338627c47650b264c477c653a67043e6766152a485c7f33617264780656537e5468143f305f4537722352303c3d4379043d69797e6f3922527b24536e310d653d4c33696c635474637d0326516f745e610d773340306621105a7361654e3e392970687c2e335f3015677d4b3a724a4659767c2f5b7c16055a126820306c14315d6b59224a27311f747f336f4d5974321a22507b22705a226c6d446a37375761423a2b5c29247163046d7e47032244377508300751727126326f117f7a38670c2b23203d4f27046a5c5e1532601126292f577776606f0c6d0126474b2a73737a41316362146e581d7c1228717664091c
```
We know, by analyzing the source code that the length of the flag is less than 38 characters and that is XOR-ed with a salt (```WeAreDe1taTeam```) and with a plaintext.

As the output is 1200 hex characters long, we know that the length of the plaintext is 600 (1200 hex characters divided by 2).

We can start by XOR-ing the output with the salt to get ```plain XOR key``` by using this [CyberChef recipe](https://gchq.github.io/CyberChef/#recipe=From_Hex%28%27Auto%27%29XOR%28%7B%27option%27:%27UTF8%27,%27string%27:%27WeAreDe1taTeam%27%7D,%27Standard%27,false%29To_Hex%28%27Space%27%29&input=NDkzODBkNzczNDQwMjIyZDFiNDIxYjMwNjAzODBjM2Y0MDNjMzg0NDc5MWIyMDI2NTEzMDY3MjExMzViNjIyOTI5NGEzYzMyMjIzNTdlNzY2YjJmMTU1NjFiMzUzMDVlM2MzYjY3MGU0OTM4MmMyOTVjNmMxNzA1NTM1NzdkM2EyYjc5MTQ3MDQwNjMxODMxNWQ3NTNmMDM2MzdmMmI2MTRhNGYyZTFjNGYyMTAyN2UyMjdhNDEyMjc1N2I0NDYwMzc3ODZhN2IwZTM3NjM1MDI0MjQ2ZDYwMTM2Zjc4MDI1NDNlNGQzNjI2NWMzZTAzNWE3MjVjNjMyMjcwMGQ2MjZiMzQ1ZDFkNjQ2NDI4M2EwMTZmMzU3MTRkNDM0MTI0MjgxYjYwN2QzMTVmNjYyMTJkNjcxNDI4MDI2YTRmNGY3OTY1N2UzNDE1M2YzNDY3MDk3ZTRlMTM1ZjE4N2EyMTc2N2YwMjEyNWIzNzU1NjM1MTdhMzc0MjU5N2I2YzM5NGU3ODc0MmM0YTcyNTA2OTYwNjU3Njc3N2MzMTQ0MjkyNjRmNmUzMzBkNzUzMDQ1M2YyMjUzN2Y1ZTMwMzQ1NjBkMjIxNDY4MzE0NTZiMWI3MjcyNWYzMDY3NmQwZDVjNzE2MTdkNDg3NTNlMjY2NjdlMmY3YTMzNGM3MzFjMjI2MzBhMjQyYzcxNDA0NTdhNDIzMjQ2MjkwNjQ0NDEwMzZjN2U2NDYyMDg2MzBlNzQ1NTMxNDM2YjdjNTE3NDNhMzY2NzRjNGYzNTJhNTU3NTQwN2I3NjdhNWM3NDcxNzYwMTZjMDY3NjM4NmU0MDNhMmI0MjM1NmE3MjdhMDQ2NjJiNDQ0NjM3NWYzNjI2NWYzZjEyNGI3MjRjNmUzNDY1NDQ3MDYyNzc2NDEwMjUwNjM0MjAwMTY2MjkyMjViNDM0MzI0MjgwMzZmMjkzNDFhMjMzODYyN2M0NzY1MGIyNjRjNDc3YzY1M2E2NzA0M2U2NzY2MTUyYTQ4NWM3ZjMzNjE3MjY0NzgwNjU2NTM3ZTU0NjgxNDNmMzA1ZjQ1Mzc3MjIzNTIzMDNjM2Q0Mzc5MDQzZDY5Nzk3ZTZmMzkyMjUyN2IyNDUzNmUzMTBkNjUzZDRjMzM2OTZjNjM1NDc0NjM3ZDAzMjY1MTZmNzQ1ZTYxMGQ3NzMzNDAzMDY2MjExMDVhNzM2MTY1NGUzZTM5Mjk3MDY4N2MyZTMzNWYzMDE1Njc3ZDRiM2E3MjRhNDY1OTc2N2MyZjViN2MxNjA1NWExMjY4MjAzMDZjMTQzMTVkNmI1OTIyNGEyNzMxMWY3NDdmMzM2ZjRkNTk3NDMyMWEyMjUwN2IyMjcwNWEyMjZjNmQ0NDZhMzczNzU3NjE0MjNhMmI1YzI5MjQ3MTYzMDQ2ZDdlNDcwMzIyNDQzNzc1MDgzMDA3NTE3MjcxMjYzMjZmMTE3ZjdhMzg2NzBjMmIyMzIwM2Q0ZjI3MDQ2YTVjNWUxNTMyNjAxMTI2MjkyZjU3Nzc3NjYwNmYwYzZkMDEyNjQ3NGIyYTczNzM3YTQxMzE2MzYyMTQ2ZTU4MWQ3YzEyMjg3MTc2NjQwOTFj). Now the output is :
```
1e5d4c055104471c6f234f5501555b5a014e5d001c2a54470555064c443e235b4c0e590356542a130a4242335a47551a590a136f1d5d4d440b0956773613180b5f184015210e4f541c075a47064e5f001e2a4f711844430c473e2413011a100556153d1e4f45061441151901470a196f035b0c4443185b322e130806431d5a072a46385901555c5b550a541c1a2600564d5f054c453e32444c0a434d43182a0b1c540a55415a550a5e1b0f613a5c1f10021e56773a5a0206100852063c4a18581a1d15411d17111b052113460850104c472239564c0755015a13271e0a55553b5a47551a54010e2a06130b5506005a393013180c100f52072a4a1b5e1b165d50064e411d0521111f235f114c47362447094f10035c066f19025402191915110b4206182a544702100109133e394505175509671b6f0b01484e06505b061b50034a2911521e44431b5a233f13180b5508131523050154403740415503484f0c2602564d470a18407b775d031110004a54290319544e06505b060b424f092e1a770443101952333213030d554d551b2006064206555d50141c454f0c3d1b5e4d43061e453e39544c17580856581802001102105443101d111a043c03521455074c473f3213000a5b085d113c194f5e08555415180f5f433e270d131d420c1957773f560d11440d40543c060e470b55545b114e470e193c155f4d47110947343f13180c100f565a000403484e184c15050250081f2a54470545104c5536251325435302461a3b4a02484e12545c1b4265070b3b5440055543185b36231301025b084054220f4f42071b1554020f430b196f19564d4002055d79
```

To crack the next level of encryption, I used the python script by Iliya Dafchev at [this address](https://idafchev.github.io/crypto/2017/04/13/crypto_part1.html). I had to modify it a little bit to have a nicer output. This script will try to find the length of the key by calculating the [Hamming distance](https://en.wikipedia.org/wiki/Hamming_distance) on blocks of the ciphertext. A better explanation is provided in the blog post of the author of the script.

The output of the script is :
```bash
$ python solve.py
Key sizes:  [30, 15, 20]
Current key size:  30
['J', 'S', 'U', 'V', 'W', '|', '}']
['3']
['l']
['`', 'c', '\x7f']
['0']
['@', 'A', 'D', 'F', 'G', 'j', 'k', 'l', 'm', 'n']
['$', '0', '3', '4', '7']
['t']
['O']
['j']
['o']
['\x1a', '\x1b', ' ', '$', ',', '-', '0', '1', '2']
[]
['u']
['5', '9']
['5']
['u']
['n']
[')', '1', '2']
['o']
['h', 'i', 'j', 'k', 'm', '|', '}', '\x7f']
['O']
['t']
['3']
['m']
['!', '%', '&', "'", '0', '1', '2', '4', '6', '7']
['H', 'I', 'c']
['l']
['\x1f', '#', '/', '0', '1', '2', '3', '4', '5', '7']
['W']
Keys to try:  0
Current key size:  15
[]
[]
[]
[]
[]
['j', 'k', 'm']
[]
['t']
[]
[]
[]
[]
[]
[]
[]
Keys to try:  0
Current key size:  20
[]
[]
[]
[]
[]
[]
[]
['l']
[]
[]
['h', 'j', 'k', 't', 'v', 'w', 'z']
[]
['l']
[]
[]
[]
[]
[]
[]
[]
Keys to try:  0
```
It founds 3 possible key length : 30, 15 and 20. As the possible key for a length of 15 and 20 are almost empty, we will focus on the length of 30 (and also because it is the most probable). The script will tell us for each character which one is the most credible.

So, I take every first characters (```J3l`0@$tOjo u55un)ohOt3m!Hl#W```) and XOR it with our cipher text. I got the following output (always using CyberChef) :
```
Tn eaDch I u! not tote thtN giih niCr eyebbFor tpe{ in eCeu | tkoXdand t<rors vove;Bue.`din mz Erart e&at loneq whae.txed dfs]~se,Wy! in d}srite ~M fixw js
gleast* to dwtg.Nor1Jru pinf Hvrs wx:h thy8tmngueqX duse geA~ghteuuNor t}nfer ftNlynz tl Ovse t~;ches hrmne,N~Y dante/ Cxr smt"l, dekipe to1Ie0isviweICo anhnsensuyl"fease.wytu tkeH7alont`But ma dive fBtc,=noq @n fivtnsensek aanDibXuqdx ome
qoolib& hearl drom bNrfisg whHr,Who1"eaves8ulswaytO dhx ljkHyess ~( a mav,Vhy pcDut ueaqtMd slag+ and naqsal fYedcu tl Or.Onlhnmy plygwe thdX vao I#cBbnt mhngain,Lhct sht.txai mbkHd me b'n awajdq me aJi~.
```
The text looks pretty good, so playing around with the other possible characters found by the python script, we finally end up with the key : ```W3lc0m3tOjo1nu55un1ojOt3m0cl3W```.

The plaintext is the sonnet 141 by Shakespeare, and the flag is ```de1tactf{W3lc0m3tOjo1nu55un1ojOt3m0cl3W}```.
