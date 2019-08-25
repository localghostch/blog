---
title: "RedpwnCTF - Web - Crypt"
date: 2019-08-16
draft: false
---

 RedpwnCTF 2019 – Every Encoding Ever

**Category :** Cryptography
**Description :** Who knew encodings could be this annoying?

## Write-up

We have a file ```out.txt```, which is 33 Ko large and contains only ASCII characters. It looks like a base64 encoded text.

So, let's open [CyberChef](https://gchq.github.io/) and use the "From Base64" operation over our file. As the output we have ASCII characteres that looks like base64 encoded, again. Let's put another "From Base64" operation. We have to repeat that 8 times to get an output that is not directly base64 encoded, there is dots between ASCII characteres (CyberChef is representing null-byte ```\x00``` as dot ```.```). This is the sign of UTF16 encoding. Let's add a "Decode Text" operation and choose "UTF16LE (1200)". Again, we have a base64 encoded text. After that, the encoding is now "UTF16BE (1201)". Finally, the last encoding we have to face is "IBM EBCDIC International (500)" (multiple times).

At the end, we got the flag : ```flag{dats_a_lot_of_charsets}```.

---

P.S. For those who wonder how I found that the encoding is "IBM EBCDIC International (500)", it's by playing with the "Magic" operation and the "Intensive mode" checked. This will "[...] attempt to detect various properties of the input data and suggests which operations could help to make more sense of it".

P.P.S The full CyberChef reicipe address is available [here](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true)From_Base64('A-Za-z0-9%2B/%3D',true)From_Base64('A-Za-z0-9%2B/%3D',true)From_Base64('A-Za-z0-9%2B/%3D',true)From_Base64('A-Za-z0-9%2B/%3D',true)From_Base64('A-Za-z0-9%2B/%3D',true)From_Base64('A-Za-z0-9%2B/%3D',true)From_Base64('A-Za-z0-9%2B/%3D',true)Decode_text('UTF16LE%20(1200)')From_Base64('A-Za-z0-9%2B/%3D',true)Decode_text('UTF16BE%20(1201)')From_Base64('A-Za-z0-9%2B/%3D',true)From_Base64('A-Za-z0-9%2B/%3D',true)From_Base64('A-Za-z0-9%2B/%3D',true)From_Base64('A-Za-z0-9%2B/%3D',true)From_Base64('A-Za-z0-9%2B/%3D',true)Decode_text('IBM%20EBCDIC%20International%20(500)')From_Base64('A-Za-z0-9%2B/%3D',true)Decode_text('IBM%20EBCDIC%20International%20(500)')From_Base64('A-Za-z0-9%2B/%3D',true)Decode_text('IBM%20EBCDIC%20International%20(500)')From_Base64('A-Za-z0-9%2B/%3D',true)From_Base64('A-Za-z0-9%2B/%3D',true)). You just have to put the "out.txt" in the input textarea.
