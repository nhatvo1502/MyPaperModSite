---
title: 'Send SMS with Twilio using AWS Lambda and API Gateway'
date: 2024-10-28T16:19:06-07:00
draft: false
---

![image](/images/twilio-microservice/twilio-microservice.drawio.png)
Welcome to my tech blog! Today, I’m excited to take you on my cloud journey with a project which I was inspired from real-world challenge: integrating SMS notifications into a software that had numerous restrictions. I needed a solution to notify users whenever the database was updated. Instead of wrestling with the software’s limitations, I decided to create a microservice that could handle SMS notifications independently. This way, the software team can invoke an API to send SMS messages without worrying about the underlying code and integration complexities.

### Setup
In this project, I use Twilio for sending text messages, combined with AWS Lambda, AWS API Gateway, and AWS Route53 for domain and API mapping. Here’s how I set it up:

1. **Twilio Setup**: I bought a number from Twilio, loaded a few bucks into the account, and created a Lambda function to send SMS whenever the API is invoked using the Twilio phone number.
2. **API Gateway Configuration**: I created a POST method API with a custom domain name and mapped it to my Lambda function.
3. **Route53 Configuration**: I updated an A record on Route53 with my API Gateway's ARN to handle the domain and API mapping seamlessly.

Check out my project on [GitHub](https://github.com/nhatvo1502/twilio-microservice)

### Improvement
There are few improvements that I would like to integrate in the future are:

1. Authentication: I could add API key authentication through custom headers or integrate Okta for more advanced identiy management. This will ensure authorized users can trigger the SMS  functionality.
2. Loggin and Monitoring: Using AWS CloudWatch would help me keep track of performance and quickly resolve any issues.
3. Reliability: Implementing retru mechanisms and error handling strategies into my lambda function would make the service more reliable, ensuring messages are sent even during temporary failures.

By the end of this project, I happy with this robust and scalable SMS notification service that can be easily integrated into any application. Let’s get started!