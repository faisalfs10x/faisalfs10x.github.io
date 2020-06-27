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

- So, here we are XD…

![enter image description here](https://raw.githubusercontent.com/faisalfs10x/faisalfs10x.github.io/master/asset/hackertest/4.png)
