# Updates of March!

Hi! I'm sharing the work done by me in the last week of March.

# Table of Content
+ How to copy table from one database to another?
+ How to convert plain text log to JSON format?

## Chapter 1: How to copy table from one database to another?

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

## Chapter 2: How to convert plain text log to JSON format?

I wanted to send the metrics data of an application to the prometheus and grafana.

## Problem statement

The application is generating logs in plain text. To send meaningful data to prometheus and grafana it is important to parse them.


## Solution
Below is the log content of the application and saved them into **opcron.txt** file
```
2020-03-30 06:20:43,012 [INFO ]  [test.py:136] Number of Files identified in the directory = 3
2020-03-30 06:20:43,012 [INFO ]  [test.py:337] Number of Files from the directory present in object storage = 3
2020-03-30 06:20:43,012 [INFO ]  [test.py:438] Number of Files successfully uploaded = 0
2020-03-30 06:20:43,013 [INFO ]  [test.py:909] Number of Files upload failed for = 0
```
Create a shell script to parse these logs and convert them into **JSON** format.\
```cat > generatejson.sh```
```
awk '{print "{\"cron_stats\": { \"completion_time\": \""$1" " $2"\", \"Task\": "}' opcron.txt > file1.txt
cat opcron.txt | cut -d']' -f3 | cut -d' ' -f2-15 | cut -d'=' -f1 > file2.txt
cat opcron.txt | cut -d']' -f3 | cut -d' ' -f2-15 | cut -d'=' -f2 > file3.txt
sed -i -e 's/[[:blank:]]*$//g' -e 's/\ /_/g' -e 's/^/"/g' -e 's/$/"/g' file2.txt
sed -i -e 's/\ / "/g' -e 's/$/"/g' -e 's/^/, "value"\:/g' -e 's/$/} }/' file3.txt
i=1
END=$(wc -l file1.txt | cut -d' ' -f1)
truncate -s 0 cronstat.txt
for  i  in  $(seq 1 $END); do var1=$(sed -n ""$i"p" file1.txt); var2=$(sed -n ""$i"p" file2.txt); var3=$(sed -n ""$i"p" file3.txt); var4="$var1$var2$var3"; i=$(expr $i + 1); echo  $var4 | tee -a cronstat.txt; done
# Remove all tmp file
rm -rf file1.txt file2.txt file3.txt
```
