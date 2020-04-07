# Updates of April!

Hi! I'm sharing the work done by me in the last week of April.

# Table of Content
+ Calculate the cost of data stored on AWS S3.


## Chapter 1: Calculate the cost of data stored on AWS S3.

Scenario: There are so many buckets which we are consuming for data stroage. Some of these buckets are costing huge amount to us. Now, we need to find out which bucket is costing more amount so we can plan for `data archive.`

## Problem statement

We need to create the excel sheet which will contain the name of the bucket and the storage occupied by the files in those bucket. The s3 service charges is **$0.25** per GB so accordingly we can apply formula in our excel sheet.

## Solution

+ Configure ```AWS CLI``` on your machine with ```ACCESS KEY and SECRET ACCESS KEY.```
+ Make sure that user have required permission if not then attach an `IAM policy.`
+ Create a shell script on your your `linux` machine. 
```
cat > s3Usage.sh
```
```
for bucket in $(aws s3 ls --profile default | awk '{print $3}')
do
bucketsize=$(aws s3 ls --summarize --recursive s3://$bucket --profile default | grep 'Total Size' | awk '{print $3}')
echo $bucket,$bucketsize >> s3usage.csv
done
echo "Your file has been saved at $PWD/s3usage.csv"
```
#### You can open csv file in your excel and update accordingly.
