---
title: 'My First Terraform'
date: 2025-02-13T17:04:35-08:00
draft: false
---
## The Goal:
The idea is to use Infrastructure as Code (IaC) to quickly spin up a simple yet secure virtual network on AWS. Terraform CLI is my tool of choice for managing it all, so I can automate the setup and ensure everything stays organized and reproducible.

## What’s Inside the Infrastructure:
- **A VPC** in the `us-west-1` region with a CIDR block of `10.0.0.0/16`—this is the foundation of the network.  
- **A Public Subnet** (`10.0.1.0/24`) connected to the internet via an Internet Gateway. The cool thing is that all the traffic from this subnet can flow freely to the internet, thanks to a simple Public Route Table.  
- **A Private Subnet** (`10.0.2.0/24`) that’s a little more closed off—this one doesn’t have direct internet access.  
- **A NAT Gateway** placed in the public subnet, with an Elastic IP. This allows instances in the private subnet to reach the internet for things like updates, but without exposing them to the outside world. We handle all this securely with a Private Route Table.

## What I Tested:
- I launched a `t3.micro` Linux instance in both the **Public Subnet** and **Private Subnet**.  
- First, I SSH into the **Public Instance** using its public IP address.  
- Then, I SSH into the **Private Instance** using its private IP address.  
- To make sure everything worked as expected, I tried a `yum update` and also pinged `google.com` from the private instance to test the internet connection.

## Key Pairs and Security Notes:
- To keep things simple, I generated a **public/private key pair** on my local computer using PowerShell:  
  ```bash  
  ssh-keygen -t rsa -b 2048 -f C:\path\to\your\keyfile  
  ```  
- I used this key pair for both the **Public** and **Private Instances**.  
- To SSH into the private instance, I created a new private key on the **Public Instance**, pasted it in, and used it for access. I know this isn’t the most secure way to handle things, but it helped me keep it simple for this demo.

## Room for Improvement:
- Going forward, it would be much better to use **AWS Secrets Manager** to securely store and manage secrets. This would help scale things more securely and take the guesswork out of key management.
