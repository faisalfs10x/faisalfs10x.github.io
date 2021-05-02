---
layout: single
title: SQL injection UNION attack on Oracle DB
excerpt: "SQL injection vulnerability in the product category filter. Then retrieve the contents of the table to obtain the username and password of all users."
date: 2020-11-11
classes: wide
header:
  teaser: 
  teaser_home_page: true
  icon: 
categories:
  - web
permalink: /wsa/SQL_injection_attack_on_Oracle
tags: [ sqli ]

---

# SQL injection UNION attack on Oracle DB

SQL injection vulnerability in the product category filter. Then retrieve the contents of the table to obtain the username and password of all users.

---
- Determine the number of columns that are being returned by the query and which columns contain text data. Then, retrieve the list of tables in the database. 

![](https://raw.githubusercontent.com/faisalfs10x/Web-Security/main/sqli/image/1.PNG)   

 - Identify the database version.
 
  ![](https://raw.githubusercontent.com/faisalfs10x/Web-Security/main/sqli/image/0.png)
  
 - Retrieve the details of the columns in the selected table. So, we have two columns in the USERS_RWNMRN table.

![](https://raw.githubusercontent.com/faisalfs10x/Web-Security/main/sqli/image/2.png)

- Eventually, retrieve the usernames and passwords in the USERS_RWNMRN table.

![](https://raw.githubusercontent.com/faisalfs10x/Web-Security/main/sqli/image/3.png)

Thanks...

