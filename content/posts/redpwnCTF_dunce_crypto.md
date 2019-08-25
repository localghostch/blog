---
title: "RedpwnCTF - Crypto - Dunce Crypto"
date: 2019-08-16
draft: false
---

RedpwnCTF 2019 – Dunce Crypto

**Category :** Crypto
**Description :** Emperor Caesar encrypted a message with his record-breaking high-performance encryption method. You are his tax collector that he is trying to evade. Fortunately for you, his crown is actually a dunce cap.

mshn{P_k0ua_d4ua_a0_w4f_tf_ahe3z}

## Write-up

This one is trivial.
As we know flag format: ```flag{}``` we can infer the key from first character: f -> m => key is 7 

Using https://www.dcode.fr/chiffre-cesar with key 7 we get the flag: flag{I_d0nt_w4nt_t0_p4y_my_tax3s}