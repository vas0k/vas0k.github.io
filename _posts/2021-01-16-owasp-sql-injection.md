---
title: "OWASP Juice Shop — SQL Injection"
description: "In this article, I am going to demonstrate an SQL injection attack on a deliberately vulnerable application that is provided by the OWASP…"
date: "2021-01-16T13:58:43.932Z"
categories: ["Tech"]
tags: ["Web Security"]
published: true
---

In this article, we are going to look at a SQL injection attack on a deliberately vulnerable application, that is provided by the OWASP foundation. This article, is for those who are interested in understanding a high level overview of the attack, and the damage that it can cause.

> **DISCLAIMER:** This attack is demonstrated only for educational purposes. I am not responsible for any legal implications that arise, if the below methods are used to exploit systems, without prior consent from the owner.

## What is SQL Injection?

SQL injection is a dangerous attack on web applications that, when exploited, could reveal sensitive information about the organization such as user account details, users present in the database, passwords, sensitive files, etc. 
The worst that could occur out of an SQL injection, is that the attacker could spawn a command shell on his/her machine, and this would enable them to laterally move around the network.

### Background

SQL injection attacks occur, due to the application improperly handling the user input (This is demonstrated in this article further below). Mitigation steps are also mentioned at the end of this article.

> Artifacts used in this article:
>
> 1. Linux machine (any distro). The below steps, can also be performed on Windows machines, but the commands would slightly differ.
> 2. Docker (For running OWASP Juice shop)
> 3. Once docker is installed, the below commands should get you up and running OWASP Juice shop. 
> 4. For pulling the latest image: `docker pull bkimminich/juice-shop`
> 5. For spinning up a container: `sudo docker run — rm -p 3000:3000 bkimminich/juice-shop`

### Exploitation steps:

#### I. Start up the Docker container

```
 sudo docker run — rm -p 3000:3000 bkimminich/juice-shop
 \ juice-shop@12.4.0 start /juice-shop
 \ node app

 info: All dependencies in ./package.json are satisfied (OK)
 info: Chatbot training data botDefaultTrainingData.json validated (OK)
 <! — snipped a part of the output— -
 info: Required file main-es2018.js is present (OK)
 info: Required file runtime-es5.js is present (OK)
 info: Port 3000 is available (OK)
 info: Server listening on port 3000
```

The application is now attached to port 3000. Let’s fire up a browser, and login to this application.

#### II. Login to the application at port 3000

![OWASP Juice Shop — Home screen!](/assets/images/owasp-juice-shop-sql-injection/asset-1.png "OWASP Juice Shop — Home screen")

This is a very simple, yet fun application, that’s written using Javascript. Feel free to play around with the various menus and pages available. For this tutorial, the login screen is what we are after.

#### III. Navigate to the Login screen

![Click on the Login button at the top right corner](/assets/images/owasp-juice-shop-sql-injection/asset-2.png)

Click on the Login button at the top right corner

#### IV. The login screen should appear.

Currently, we don’t have any login credentials. However, this web application is vulnerable to SQL injection attacks. Let’s take a look, at how to exploit this.

![](/assets/images/owasp-juice-shop-sql-injection/asset-3.png)

#### V. On the login screen, type the below string in the “Email” field. Also, type any password you like. For the below example, the password “test” is used.

` test’ OR 1=1;- - `

Hit Log In.

![Payload](/assets/images/owasp-juice-shop-sql-injection/asset-4.png)

#### VI. The application logged into the home screen. Also, the current user is “admin”. This shows, that we were able to bypass existing authentication mechanisms, and login as the administrator of the application.

![Exploit Succesfull](/assets/images/owasp-juice-shop-sql-injection/asset-5.png)

## How did this attack work?

A regular SQL query looks like the following:

```
SELECT * from user_table WHERE user = ‘test’ AND pass = ‘pass’
```
This SQL query basically says “ Give me the user from the user_table where the user is “test” AND the password is “pass”.

From the input that we provided above, the SQL query would look like:

```
SELECT * from user_table WHERE user = ‘test’ OR 1=1;- — AND pass = ‘test’
```
This SQL query says “ Give me the user from the user_table where the user is “test” OR 1=1. The application looked at the first record in the DB, and since 1=1 always evaluates to true, the application allowed access. Since we logged in as admin, we can assume that the first record in the DB was for the admin user.

## How to prevent this attack?

There are 2 ways to prevent this attack:
1. Sanitize user input. If you are a developer, always check user input for any strange characters, that typically won’t be used in any application.
2. Use parameterized queries — The SQL query is pre-compiled, and the only thing left to provide to the query are the parameters (username and password in our case). This prevents the username and password, from directly being appended, into the query.

Hope you enjoyed this article.
