---
title: 'Shopify Daily Report'
date: 2024-11-04T14:45:48-07:00
draft: false
---

![image](/images/shopify-api-daily-report/cover.png)
Hey there! Welcome to my tech blog. I’m excited to share a little project: automating Shopify orders data into a database warehouse and setting up the connection for analysis. I wanted a simple way to fetch data and get real-time insights without all the manual effort. So, I set up AWS Event Scheduler to trigger data retrieval, used AWS Lambda for serverless processing, and chose DynamoDB for storage. To connect DynamoDB to Microsoft Power BI, I used CDATA Universal Software to create an ODBC driver.

I'm sure there are more sophisticated solutions out there and this is my humble approach, but I hope you find this useful! Let’s turn Shopify orders into awesome business insights together!
![image](/images/shopify-api-daily-report/bi4.png)

### Setup

1. AWS Lambda: I created a Lambda function to fetch data from Shopify and store it in DynamoDB.
2. AWS Event Scheduler: I use Event Scheduler to trigger the Lambda function daily.
3. AWS DynamoDB: While a relational database might be better for a data warehouse, I opted for DynamoDB for its simplicity and cost-effectiveness.
4. CDATA Amazon DynamoDB ODBC Driver: I configured my DynamoDB ODBC driver with CDATA since they offer free trials. This driver is essential for pulling data into Power BI since it doesn’t support DynamoDB out-of-the-box.
5. Power BI: Finally, I set up the ODBC driver as a data source in Power BI to pull in my data and start visualizing!
6. CloudWatch: I also set up CloudWatch for logging to keep track of everything.

### Improvements
1. Explore a Relational Database: Consider using Amazon RDS instead of DynamoDB for enhanced data management and querying capabilities, leading to more efficient analysis.
2. Improve Error Handling: Implement more robust error handling in the AWS Lambda function to ensure better resilience during data retrieval.
3. Flexible Scheduling: Adjust the data retrieval schedule based on business needs, rather than sticking to a fixed daily trigger, to optimize resource usage.
4. Data Validation and Transformation: Incorporate processes for data validation and transformation before loading data into DynamoDB to improve overall data quality.
5. Enhance Power BI Dashboards: Add more interactive elements and deeper analytics in Power BI to provide valuable insights for decision-making.

Check out this project on my [GitHub](https://github.com/nhatvo1502/twilio-microservice).

Thank you for visiting!