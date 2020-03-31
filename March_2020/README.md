# Updates of March!

Hi! I'm sharing the work done by me in the last week of March.

# Table of Content
+ How to copy table from one database to another.
+ How to convert plain text log to JSON format.

## Chapter 1: How to copy table from one database to another

Scenario: I'm trying to build a Jenkins pipeline. According to the requirement, the pipeline should take backup the content from a database and restore it to the new **mysql** database running on `Kubernetes` 

## Problem statement

When I have tried to restore the dump to new `mysql server`. It was creating a database which was backed up during the dump process. In a moment, I realised that my application is trying to hit another database for the content. In this case, I left with two options: 
* Update the code of the application
* Create a database required by application and copy all the tables from one database to another.

## Solution

Let's assume that the name of the dump file is `demo.sql` which has a database named `prod-thor`
* Take the dump of mysql database
```mysqldump -h remote-db-url -u root -pYourDbPassword --databases prod-thor > demo.sql```
* Update the name of the database 
```sed -i 's/prod-thor/demo-thor/g' demo.sql```
* Create a database with the name of `demo-thor` in mysql server
```mysql -h 127.0.0.1 -u root -pYourDbPassword -e CREATE DATABASE demo-thor```
* Copy all tables from one database to another
```mysql -h 127.0.0.1 -u root -pYourDbPassword -e CREATE TABLE `newdb`.table1 SELECT * FROM olddb.tb1;```
