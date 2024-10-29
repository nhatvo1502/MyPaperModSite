---
title: 'Send SMS with Twilio using AWS Lambda and API Gateway'
date: 2024-10-28T16:19:06-07:00
draft: false
author: Nhat Vo
---

![image](/images/twilio-microservice/twilio-microservice.drawio.png)
Welcome to my tech blog! Today, I’m excited to share a project that came about from a real-world need: integrating SMS notifications into software that had numerous restrictions and challenges. I needed a solution to notify users whenever the database was updated. Instead of wrestling with the software’s limitations, I decided to create a microservice that could handle SMS notifications independently. This way, the software team can invoke an API to send SMS messages without worrying about the underlying code and integration complexities.

In this project, I use Twilio for sending text messages, combined with AWS Lambda, AWS API Gateway, and AWS Route53 for domain and API mapping. Here’s how I set it up:

1. **Twilio Setup**: I bought a number from Twilio, loaded a few bucks into the account, and created a Lambda function to send SMS whenever the API is invoked using the Twilio phone number.
2. **API Gateway Configuration**: I created a POST method API with a custom domain name and mapped it to my Lambda function.
3. **Route53 Configuration**: I updated an A record on Route53 with my API Gateway's ARN to handle the domain and API mapping seamlessly.

By the end of this tutorial, you’ll have a robust and scalable SMS notification service that can be easily integrated into any application. Let’s get started!