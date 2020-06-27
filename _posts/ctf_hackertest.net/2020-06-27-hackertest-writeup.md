---
layout: single
title: "HackerTest.net write-up"
date: 2020-06-27 00:22:00 -0000
classes: wide
categories: ctf
permalink: /ctf/HackerTest
tags: [ctf, web, javascript, php]

---

HackerTest.net is your own online hacker simulation. 
With 20 levels that require different skills to get to another step of the game, this new real-life imitation will help you advance your security knowledge.
HackerTest.net will help you improve your JavaScript, PHP, HTML and graphic thinking in a fun way that will entertain any visitor!
Have a spare minute? Log on! Each level will provide you with a new, harder clue to find a way to get to another level.
Will you crack HackerTest.net?_

## Level 1
- Level 1 as starting point to the next 20 levels in this challenge.
![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/hackertest/1.png)

- If we inspect the code at the form, we will get `javascript:check()` action.
![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/hackertest/2.png)

- So, let jump to `check()` function and we get `var a = “null”`. If we input “`null`” as the password, it will bring us to `http://www.hackertest.net/null.htm`, else it will popup alert as “`Try again`” as below.
![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/hackertest/3.png)

- Here we are XD.
- :(
![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/hackertest/4.png)

## Level 2

- In level 2, just inspect the `null.htm` as before. Reviewing the javascript code, the `pass==”l3l”` and it will bring us to next level.
![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/hackertest/5.png)

## Level 3

![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/hackertest/6.png)

- For level 3, in js code there a method called `String.fromCharCode()`. The `fromCharCode()` method converts Unicode values into characters.
**Note:** This is a static method of the String object, and the syntax is always `String.fromCharCode()`. Then, just simply decode the Unicode values online.
![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/hackertest/7.png)

- There, we get “`abrae`” as output. then just go to `http://www.hackertest.net/abrae.htm` to next level.
![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/hackertest/8.png)

## Level 4

![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/hackertest/9.png)

- In level 4, clicking the “`Click here`” will bring us to `sdrawcab.htm`, so let us inspect it.
![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/hackertest/10.png)

- As easy as l0l, it just giving us the password for level 5.
![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/hackertest/11.png)

## Level 6

![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/hackertest/12.png)

- As for level 6, the script will check the password from `psswd.js` file as below.
![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/hackertest/13.png)

- By refering to `psswd.js` lead us to `var pass` as below. Voilla..
![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/hackertest/14.png)

## Level 7

- In this level, we get login page. It is an authentication challenge.
![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/hackertest/15.png)

- By login with random credential, it brings us to `pwd2.php` site. After inspecting it, there’s nothing interesting there.
![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/hackertest/16.png)

- Let come back to main page. Walaoo, there’s nothing about javascript here but there is only one thing that caught my eyes. What’s that? It is `included.gif` hehe..By seeing with only one eye, we don’t think it is suspicious right? I’m stuck here for 3 minutes hahahaha… 
![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/hackertest/17.png)

- After browsing to `site/images/included.gif` route us to a big white image. Do you see what I see?? Once again, please look at the gif with ***4x optical zoom*** of your eyes hahaha.. Damn..super ~~tiny credential~~ at the right bottom there… almost ignore the tiny black credential there.
-  *It’s like our life. We only focus things in front of our eyes. We don’t see the good sides of people at whole things instead of pointing to their mistakes at only one single place.*
![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/hackertest/18.png)
