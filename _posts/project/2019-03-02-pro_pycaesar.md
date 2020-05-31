---
layout: single
title: "Cryptography-Caesar-Cipher-with-Python"
excerpt: "I got a task to make a simple script to brute force Caesar Cipher using Python during my class HACKING-TECHNIQUES-AND-PREVENTION project."
date: 2019-03-02 00:22:00 -0000
classes: wide
categories: tool
permalink: /pro_pycaesar
tags: [tool, python, caesar, decryption]

---

I got a task to make a simple script to brute force Caesar Cipher using Python during my class HACKING-TECHNIQUES-AND-PREVENTION project.

## Brute Force Caesar Cipher using Python:
- Write a python program to decrypt the following Caesar ciphertext.

MVGRZUDPULVJKZDVRWKVIKZDVZMVUFEVDPJVEKVETVSLK TFDDZKKVUEFTIZDVREUSRUDZJKRBVJZMVDRUVRWVNZMV YRUDPJYRIVFWJREUBZTBVUZEDPWRTVSLKZMVTFDVKYIFL XYNVRIVKYVTYRDGZFEJDPWIZVEUJREUNVCCBVVGFEWZXY KZEXKZCKYVVEUNVRIVKYVTYRDGZFEJNVRIVKYVTYRDGZF EJEFKZDVWFICFJVIJTRLJVNVRIVKYVTYRDGZFEJFWKYVNFI CUZMVKRBVEDPSFNJREUDPTLIKRZETRCCJPFLSIFLXYKDVW RDVREUWFIKLEVREUVMVIPKYZEXKYRKXFVJNZKYZKZKYRE BPFLRCCSLKZKJSVVEEFSVUFWIFJVJEFGCVRJLIVTILZJVZTF EJZUVIZKRTYRCCVEXVSVWFIVKYVNYFCVYLDREIRTVREUZR ZEKXFEERCFJVNVRIVKYVTYRDGZFEJDPWIZVEUJREUNVCC

## Plain text after decrypted:

PLAIN TEXT : VE PAID MY DUES TIME AFTER TIME I VE DONE MY SENTENCE BUT COMMITTED NO CRIME AND BAD MISTAKES I VE MADE A FEW I VE HAD MY SHARE OF SAND KICKED IN MY FACE BUT I VE COME THROUGH WE ARE THE CHAMPIONS MY FRIENDS AND WELL KEEP ON FIGHTING TIL THE END WE ARE THE CHAMPIONS WE ARE THE CHAMPIONS NO TIME FOR LOSERS CAUSE WE ARE THE CHAMPIONS OF THE WORLD I VE TAKEN MY BOWS AND MY CURTAIN CALLS YOU BROUGHT ME F AME AND FORTUNE AND EVERYTHING THAT GOES WITH IT I THANK YOU ALL BUT ITS BEEN NO BED OF ROSES NO PLEASURE CRUISE I CONSIDER IT A CHALLENGE BEFORE THE WHOLE HUMAN RACE AND I AINT GONNA LOSE WE ARE THE CHAMPIONS MY FRIENDS AND WELL

## Output as below:
![Screenshot](https://raw.githubusercontent.com/faisalfs10x/HACKING-TECHNIQUES-AND-PREVENTION-cryptography-caesar-cipher/master/evidence.png)

## Source code:
- [Get full source code here](https://github.com/faisalfs10x/HACKING-TECHNIQUES-AND-PREVENTION-cryptography-caesar-cipher/blob/master/testBrute.py)

        # Reference
        # https://inventwithpython.com/hacking/chapter7.html
        # python 2.7


        print "Hai, saya Faisal dan ini utk lab3"
        mesejOrgtu = raw_input("masukkan text yg telah dicipherkan baris demi baris: ")

        kombinasihuruf = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'

        # loop pada semua kunci yg possible
        for key in range(len(kombinasihuruf)):

        # It is important to set translated to the blank string so that the
        # previous iteration's value for translated is cleared.

          translated = ''

        # The rest of the program is the same as the original Caesar program:
        # run the encryption/decryption code on each symbol in the message

          for symbol in mesejOrgtu:
            if symbol in kombinasihuruf:
              num = kombinasihuruf.find(symbol) # dptkan nombor pd symbol
              num = num - key

        # handle the wrap-around if num is 26 or larger or less than 0

              if num < 0:
                num = num + len(kombinasihuruf)

        # tambah nombor simbol tu pada hujung yg dah diterjemah

              translated = translated + kombinasihuruf[num]

            else:
        # hanya tambah simbol tanpa encrypt atau decrypt

              translated = translated + symbol

        #paparkan kunci yg dah di test sekali dgn decryption dia.
        #kat sini kita kena cari sendiri text yang valueable atau ade makna meaningful.

          print('Key #%s: %s' % (key, translated))
