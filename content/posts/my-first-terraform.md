---
title: 'My First Terraform'
date: 2025-02-13T17:04:35-08:00
draft: false
---
![image](/images/my-first-terraform/my-first-terraform.drawio.png)

Check out my project on [GitHub](https://github.com/nhatvo1502/my-first-terraform)
## The Goal:
The idea is to use Infrastructure as Code (IaC) to quickly spin up a simple yet secure virtual network on AWS. Terraform CLI is my tool of choice for managing it all, so I can automate the setup and ensure everything stays organized and reproducible.

## What’s Inside the Infrastructure:
- **A VPC** in the `us-west-1` region with a CIDR block of `10.0.0.0/16`—this is the foundation of the network.  
- **A Public Subnet** (`10.0.1.0/24`) connected to the internet via an Internet Gateway. The cool thing is that all the traffic from this subnet can flow freely to the internet, thanks to a simple Public Route Table.  
- **A Private Subnet** (`10.0.2.0/24`) that’s a little more closed off—this one doesn’t have direct internet access.  
- **A NAT Gateway** placed in the public subnet, with an Elastic IP. This allows instances in the private subnet to reach the internet for things like updates, but without exposing them to the outside world. We handle all this securely with a Private Route Table.

## Testing:
- I launched a `t3.micro` Linux instance in both the **Public Subnet** and **Private Subnet**.  
- First, I SSH into the **Public Instance** using its public IP address.
![image](/images/my-first-terraform/sshpublic.png)
- Then, I recreated my private key and stored it on the Public Instance, before SSH into the **Private Instance** using its private IP address.
![image](/images/my-first-terraform/sshprivate.png)
- From Private Instance, I couldn't ping google.com nor able to `yum update it`. The first thing I check was my Route Tables and voila, looks like I forgot to associate my Private RT with Private subnet, therefore AWS has to created a default RT and associated it with the Private Subnet which is not what we want because our instance cannot reach NAT Gateway therefore no connection.
![image](/images/my-first-terraform/routetablemessup.png)
- I went back to my main.tf and added this block...
```
# Associate Private Route Table with Private Subnet
resource "aws_route_table_association" "private" {
	subnet_id = aws_subnet.terraform_private_subnet_1.id
	route_table_id = aws_route_table.terraform_rt_private_1.id
}
```
...rerun terraform...
```
terraform plan
terraform apply --auto-approve
```
...make sure Private Route Table is in placed, as you can see, the Private Subnet is now properly...
![image](/images/my-first-terraform/routetablecorrected.png)
... I can ping and update Private Instance as the traffic has succesfully flows through the Private Route Table and reach NAT Gateway!
![image](/images/my-first-terraform/testconnection1.png)
![image](/images/my-first-terraform/testconnection2.png)


## Key Pairs and Security Notes:
- To keep things simple, I generated a **public/private key pair** on my local computer using PowerShell:  
  ```bash  
  ssh-keygen -t rsa -b 2048 -f C:\path\to\your\keyfile  
  ```  
- I used this key pair for both the **Public** and **Private Instances**.  
- To SSH into the private instance, I created a new private key on the **Public Instance**, pasted it in, and used it for access. I know this isn’t the most secure way to handle things, but it helped me keep it simple for this demo.

## Room for Improvement:
- Going forward, it would be much better to use **AWS Secrets Manager** to securely store and manage secrets. This would help scale things more securely and take the guesswork out of key management.
