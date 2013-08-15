---
layout: post
title: "Ubuntu Install Mysql Note"
description: ""
category: “development envirment”,"database"
tags: [Mysql,database]
---
{% include JB/setup %}

Recently,I am learning server and web development.So I need to install a database on my ubuntu 
system and I choosed mysql for my first try.Now I record the whole process I install Mysql server and client.


### Step 1
Check whether  PC has installed Mysql.Simply,type `mysql` in the terminal shell,you will see.


### Step 2
You don't need come to this step if you find mysql in step 1 above.

To install mysql on ubuntu,just type the following commad :


                sudo apt-get install mysql-server mysql-client


It will do everything for you.During the installation,you need to comfirm it by type "y" and set the password for root user
You can type "mysql" to identify whether the installation was success.


When installed,You need some basic operations.

#### Lauch/Stop
Check whether the Mysql progress lauch ,type the following command if lauched it will return the id,otherwise nothing happen.

                pgrep mysqld

Lauch :

                sudo start mysql 

Stop:

                sudo stop mysql

#### Login
mysql -u username -p

-u: the login username ; -p :whether use password.

                mysql -u root -p # login with root user and you need to type root pwd .


#### Create database
                create database  < databasename > ;

#### Show database
                show databases;

#### Delete database
                drop database < databasename >;

#### Connect database
                use < databasename >; #connect to databasename if exist

#### Show current databse
                select database(); #show the current use/connect database

#### Show Tables
                show tables;

#### Change Password
                mysqladmin -u < username > -p < oldpassword > < newpassword >

#### Add User
                grant < rights > on < databasename > to < username@host> identified by < "password" >

