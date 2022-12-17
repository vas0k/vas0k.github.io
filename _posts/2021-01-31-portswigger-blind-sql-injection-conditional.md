---
title: "Blind SQL Injection — Conditional Errors"
description: "In this article, I will be exploiting a Blind SQL Injection vulnerability, on a vulnerable web application, that is hosted at Port Swigger…"
date: "2021-01-31T13:29:54.623Z"
categories: []
published: true
---

## Introduction

In this article, I will be exploiting a Blind SQL Injection vulnerability, on a vulnerable web application, that is hosted at Port Swigger Web Academy.

 **Disclaimer**: As with all things related to ethical hacking, this machine is an intentionally vulnerable machine, whose purpose is to learn ethical hacking techniques. The techniques mentioned should NOT be used for illegal purposes, and should NOT be used on machines without prior authorization from the machine owner.

### What is a Blind SQLi?

SQL injections take advantage of applications, that don’t process user input correctly. This enables the attacker, to send malicious SQL query strings, that will be processed by the underlying database, in a way that was never intended. 
In order for an attacker to know if this vulnerability exists, the web application should ideally behave differently based on the request he/she sends. However, if the web application’s response doesn’t show any difference, regardless of the request, then it becomes difficult for an attacker to know if this vulnerability exists. 
This is where we use a blind SQL injection. We typically try to get a different response from the web application, based on a particular condition. If this condition is TRUE, the response is loaded correctly. If this condition is FALSE, the web application throws an error.

The goal of this lab, is to exploit this vulnerability, to grab the administrator password, and login to the application. I had already solved this lab, but will be doing the same again, for the purpose of this article.

![Lab Description!](/assets/images/blind-sql-injection-conditional-errors/asset-1.png "Lab Description")

## Methodology

Access the lab, and intercept a request from the web browser. I have already intercepted the request, using Burp Suite. Send this request to “_Repeater_”.

![Send to repeater!](/assets/images/blind-sql-injection-conditional-errors/asset-2.png "Send to repeater")

 Repeater is a functionality in Burp Suite, that enables us to view the requests and responses in real-time, without having to reload the page every time.

The request should look like this:

![Request](/assets/images/blind-sql-injection-conditional-errors/asset-3.png "Request")

As mentioned in the lab description, the vulnerability exists in the “_TrackingId_” parameter of this request. This ID is used directly in an SQL query. An attacker, can replace this TrackingID, with parts of an SQL query, which would cause the application to behave in a manner that was never intended.

Ideally, we would check if the vulnerability actually exists using other techniques, that are beyond the scope of this article. As we already know the vulnerability exists in this parameter, let’s go ahead and add some malicious input.

Let’s modify the request, and send it to the web application.

![Modify the request and send it](/assets/images/blind-sql-injection-conditional-errors/asset-4.png "Modify the request and send it")


 The line **test’ OR 1=1--** tries to bypass existing cookie restrictions, and login to the application. Since 1=1 always evaluates to true, the condition succeeds at the back-end database side, and the attacker logs in.

After sending this request, we get a 200 OK response. However, if you look at the response, there is no indication that we successfully logged in, or whether the above SQL query returned any records.

Let’s try another technique here.

![Blind SQL injection payload, to trigger a response based on a condition](/assets/images/blind-sql-injection-conditional-errors/asset-5.png "Blind SQL injection payload, to trigger a response based on a condition")


 The line **test’ OR SELECT CASE…** checks for a condition, and based on that condition, the output would result in either ‘1’ or 1/0. Since 1/0 would result in a divide by zero exception, we should see that error on the UI side. The above condition is 1=1, which is true. Using this technique, we can check whether the application behaves differently or not.

The above query resulted in a 200 OK response. Let’s try the condition 1=2 (FALSE).

![1=2 condition, triggers an error.](/assets/images/blind-sql-injection-conditional-errors/asset-6.png "1=2 condition, triggers an error")

We see that the condition 1=2 triggers an error. It looks like the application didn’t handle the error properly, which we can leverage.

Let’s alter the condition, to check for the first character of the administrator’s password, in the SQL database.

![Alter the condition, to brute force the password in the database](/assets/images/blind-sql-injection-conditional-errors/asset-7.png "Alter the condition, to brute force the password in the database")

 This query checks for the first character of the administrator’s password. If this character is greater than ‘a’, the condition would result in TRUE. Else, the condition is false.

Since the response gave a 200 OK, we now know, that the first character is greater than ‘a’. If we try to check, if the condition is lesser than 0, the application would throw an error.

![The condition fails, so the error is displayed](/assets/images/blind-sql-injection-conditional-errors/asset-8.png "The condition fails, so the error is displayed")

Similarly, we can manually “brute-force” each character, or we can use another tool, called “Turbo Intruder”, that does this in an automated fashion. We can also use the in-built Burp Intruder, however, as this is a Community Edition, we don’t have the functionality to configure the number of concurrent requests, that we can send.

Send the request to Turbo Intruder.

![Send to Turbo Intruder](/assets/images/blind-sql-injection-conditional-errors/asset-9.png "Send to Turbo Intruder")

Turbo Intruder looks like this:

![Turbo Intruder](/assets/images/blind-sql-injection-conditional-errors/asset-10.png "Turbo Intruder")

Replace the line

`SUBSTR((SELECT password from users WHERE username=’administrator’),1,1)<’a’`

with

`SUBSTR((SELECT password from users WHERE username=’administrator’),%s,1)=’%s’`

The %s is just a placeholder, to instruct Turbo Intruder, to replace with the input that we specify.

We need to specify the input that would be used, in place of this placeholder. Turbo Intruder takes the values that you specify from the file path, that you specify (see below image).

![Path to the input/payload files](/assets/images/blind-sql-injection-conditional-errors/asset-11.png "Path to the input/payload files")

The content of those two files, in my case, is as follows.

```
 (kali㉿kali)-[~/web-academy/1.sql-injection]
 └─$ cat one 
 1
 2
 3
 4
 5
 6
 7
 8
 9
 10
 11
 12
 13
 14
 15
 16
 17
 18
 19
 20


 (kali㉿kali)-[~/web-academy/1.sql-injection]
 └─$ cat two
 a
 b
 c
 d
 e
 f
 g
 h
 i
 j
 k
 l
 m
 n
 o
 p
 q
 r
 s
 t
 u
 v
 w
 x
 y
 z
 0
 1
 2
 3
 4
 5
 6
 7
 8
 9
 A
 B
 C
 D
 E
 F
 G
 H
 I
 J
 K
 L
 M
 N
 O
 P
 Q
 R
 S
 T
 U
 V
 W
 X
 Y
 Z

```

Let’s alter the way the application shows us the results. We want only those results, that don’t trigger an error (in other words, responses that are not a 500 server error). The code, to do that would be :

![Only show successful responses](/assets/images/blind-sql-injection-conditional-errors/asset-12.png "Only show successful responses")

Run the exploit.

![Attack is running](/assets/images/blind-sql-injection-conditional-errors/asset-13.png "Attack is running")

Let it complete, and we should have our password.

![The payload column, shows us the password. Take note of all the corresponding values](/assets/images/blind-sql-injection-conditional-errors/asset-14.png "The payload column, shows us the password. Take note of all the corresponding values")

The values are shown in the format:

`<first payload/<second payload`

Where first payload, and second payloads correspond to the %s, that we saw earlier. 
If we take all the second payloads together, we get the below value (the administrator password).

 > 4veoivo0wnolzr2ai0xx

Let’s login to the application, and see if it works.

![](/assets/images/blind-sql-injection-conditional-errors/asset-15.png "")

![IT WORKS!](/assets/images/blind-sql-injection-conditional-errors/asset-16.png "IT WORKS!")

We have successfully logged into the application, using the administrator password that was retrieved, using Blind SQL injection.
