---
layout: single
title: Extracting administrator credential via blind SQL injection in cookie header
excerpt: "Blind SQL injection vulnerability in the cookie header. Able to retrieve the contents of the table to obtain the username and password of administrator."
date: 2020-11-20
classes: wide
header:
  teaser: 
  teaser_home_page: true
  icon: 
categories:
  - web
permalink: /wsa/blind SQL injection
tags: [ sqli ]

---

# Extracting administrator credential via blind SQL injection in cookie header

Blind SQL (Structured Query Language) injection is a method of SQL Injection attack in which the attacker asks the database true or false questions and then decides the answer based on the application's response.
This attack is often used where the web application is designed to display generic error messages but has not mitigated the SQL injection-vulnerable code. 

This post is about blind SQL injection vulnerability in the cookie header that result if retrieving the contents of the table to obtain the username and password of administrator. 

---
- To identify the blind SQL injection vulnerability, a simple boolean payload such `' AND '1'='1` could be used to test whether specific message appears in the response. For example, the server return `Welcome back!` message when the condition is true.

 ![](https://raw.githubusercontent.com/faisalfs10x/Web-Security/main/sqli/image/blind1.PNG)   
 
 - While `' AND '1'='2` which is false condition, the server doesn't return `Welcome back!` message.
 
  ![](https://raw.githubusercontent.com/faisalfs10x/Web-Security/main/sqli/image/blind2.PNG)
- To enumerate table name of `users` , the query can be used as shown below. The server return `Welcome back!` message that indicate the database has `users` table.

![](https://raw.githubusercontent.com/faisalfs10x/Web-Security/main/sqli/image/blind3.PNG)
- To enumerate username administrator if exist, the query can be used. Hence, we know that the database contain administrator username.

![](https://raw.githubusercontent.com/faisalfs10x/Web-Security/main/sqli/image/blind4.PNG)
- To further, we can enumerate the password length using the query using burp intruder. The query below is used to test whether the password length is having more than 20 characters or not. Unfortunately, the welcome back message doesn't appear that result the password length is likely below 21.

![](https://raw.githubusercontent.com/faisalfs10x/Web-Security/main/sqli/image/blind5.PNG)
- Then, we can test the length for more than 19 as query below that indicate that the password length is more than 19 which is 20 in length.

![](https://raw.githubusercontent.com/faisalfs10x/Web-Security/main/sqli/image/blind6.PNG)
- So, we know that the password length is 20 characters. Hence, we can enumerate the password using the query. This uses the `SUBSTRING()` function to extract a single character from the password, and test it against a specific value. In burp intruder, place payload position markers around the final `a` character in the cookie value. 

![](https://raw.githubusercontent.com/faisalfs10x/Web-Security/main/sqli/image/blind7.PNG)
- To test the character at each position, you'll need to send suitable payloads in the payload position that you've defined. You can assume that the password contains only lowercase alphanumeric characters. Go to the Payloads tab, check that "Simple list" is selected, and under "Payload Options" add the payloads in the range a - z and 0 - 9. You can select these easily using the "Add from list" drop-down.

![](https://raw.githubusercontent.com/faisalfs10x/Web-Security/main/sqli/image/blind8.PNG)
- To be able to tell when the correct character was submitted, you'll need to grep each response for the expression "Welcome back". To do this, go to the Options tab, and the "Grep - Match" section. Clear any existing entries in the list, and then add the value "Welcome back". Launch the attack by clicking the "Start attack" button or selecting "Start attack" from the Intruder menu.

![](https://raw.githubusercontent.com/faisalfs10x/Web-Security/main/sqli/image/blind9.PNG)
-   Review the attack results to find the value of the character at the first position. You should see a column in the results called "Welcome back". One of the rows should have a tick in this column. The payload showing for that row is the value of the character at the first position which is `c` character.

![](https://raw.githubusercontent.com/faisalfs10x/Web-Security/main/sqli/image/blind10.PNG)
- Now, you simply need to re-run the attack for each of the other character positions in the password, to determine their value. To do this, go back to the main Burp window, and the Positions tab of Burp Intruder, and change the specified offset from 1 to 2. You should then see the following as the cookie value: `TrackingId=???????' AND (SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator')='$a$`

![](https://raw.githubusercontent.com/faisalfs10x/Web-Security/main/sqli/image/blind11.PNG)
- Launch the modified attack, review the results, and note the character at the second offset which is `1` character.

![](https://raw.githubusercontent.com/faisalfs10x/Web-Security/main/sqli/image/blind12.PNG)
- Continue this process testing offset 3, 4, and to the last 20, until you have the whole password which is `c1jtXXXXXXXXXXy3hbp9` that having 20 characters long. 

Reference:

- https://owasp.org/www-community/attacks/Blind_SQL_Injection
- https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses
