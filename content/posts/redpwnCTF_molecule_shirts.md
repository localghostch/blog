---
title: "RedpwnCTF - Forensics - Molecule Shirts"
date: 2019-08-16
draft: false
---

RedpwnCTF 2019 – Molecule Shirts

**Category :** Forensics
**Description :** Apparently, this picture has a name? The flag is in the format flag{name}.

## Write-up

We were given a picture of a big chemical molecule.
Doing image search on google leads me to this website: http://www.molecularshirts.com/
Hum, same name as the challenge name, We are on the right way

There is a section on the site explaining how to form the molecule: http://www.molecularshirts.com/dont-get-it/

The description of the challenge is saying "this picture has a name"... Let's look at the "your name in molecules!" section: http://www.molecularshirts.com/your-name-in-molecules/

From here, We know that the first letter of the name is drawn on the left. So let's check A,B,C... until we have a match between the left part of our image and the left part of the name...

After some tests, the molecule is a ```DRAMSTRON``` and the associated name is ```DR.ARMSTRONG```

flag: flag{DR.ARMSTRONG}

