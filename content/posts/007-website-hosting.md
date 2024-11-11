---
title: 'How I host my website on AWS using Domain from Godaddy.com'
date: 2024-01-29T11:13:33-08:00
draft: false
author: Nhat Vo
---

# Intro
Hey there! When I decided to launch my website, I wanted a reliable and scalable solution. After buying my domain from GoDaddy, I turned to Amazon Web Services (AWS) for hosting. It’s been an exciting journey figuring out the best setup.

I use Amazon S3 to store my website files, Route 53 for domain name system (DNS) management, and CloudFront to deliver content quickly and securely. It wasn’t all smooth sailing, but the flexibility and power of AWS made it worth it. Join me as I share the highs and lows of getting my site online using GoDaddy and AWS services!

![image](/images/007/Drawing8.png)

# Step 1: Purchase my first domain
Initially, I planned to buy my domain directly from AWS, but Route 53 doesn’t support the .one top-level domain. So, I ended up purchasing it from GoDaddy instead. This choice means I need to update the Nameserver (NS) records in the GoDaddy console—an extra step compared to buying directly from AWS, where this would be unnecessary.

If you're curious about which top-level domains AWS supports, you can check out the full list here[https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/registrar-tld-list.html].

Having a domain on GoDaddy and hosting on AWS means I have to manage the Nameservers accordingly.

# Step 2: Route53 NameServers

First, I need to create a Route 53 Hosted Zone. AWS will provide four NameServers (NS) by default. I’ll use these to update the NS records on my GoDaddy domain.

## Creating a New Hosted Zone in Route 53
When you buy a domain from GoDaddy, it comes with two default NS records pointing to GoDaddy servers. Since I will need to configure CloudFront in the end, I will have Godaddy gives up DNS administration to AWS Route 53.

### Steps to Generate New AWS NameServers from Route 53:
1. Sign in to **AWS Management Console** and navigate to Route53.
2. On the left panel, select **Hosted zones**.
3. Choose **Create hosted zone**.
4. Under **Domain name**, I entered nvo.one
5. Under **Type**, I left it unchanged as **Public hosted zone**.
6. Under **Tags**, choose **Add tag**, with **Key** is **"Website"** and **Value** is **"www.nvo.one"*** to keep track of my resources later on.
7. Choose **Create Hosted Zone**
8. Route 53 will automatically generate a new set of NS records (four in total). Keep this tab open, as you'll need these values for the next step.

## Update GoDaddy Domain's NS
Next, I need to update my GoDaddy domain’s NS records to use the AWS NameServers.

### Steps to Update GoDaddy Domain's NS Records:
1. Sign in to `Godaddy.com` console
2. Choose the "Nine dots" next to my Profile to expand the nav menu.
3. Choose `Domains`
4. Highlight my domain name, choose the "3 dots" to expand the settings bar and choose `Edit DNS`
5. Under my domain `Domain Settings` menu bar, choose `Nameservers`
6. Choose `Change Nameservers`
7. Confirm the change by choosing `Yes` to replace the GoDaddy NS records.
8. Go back to your AWS Route 53 tab to copy each NS record and paste them into the GoDaddy console. By default, GoDaddy provides only two NS records, so you’ll need to add two more to match the four provided by AWS Route 53.

# Step 3: Request a SSL certificate

My website requires a public certificate so Amazon CloudFront can enforce HTTPS, ensuring all connections between CloudFront and viewers are encrypted.

However, CloudFront can only use certificates created in the US East (N. Virginia) region. If an SSL certificate is created in any other region, CloudFront won't be able to use it. Here’s how I create my free AWS-provided SSL certificate:

1. Sign in to **AWS Management Console** and navigate to **ACM**
2. On the left panel, select **Request certificate**
3. Keep the default selection **Request a public certificate*, then choose **Next**
4. In the **Domain names** section, I entered my domain name **nvo.one**
5. In the **Validation method**, I choose **DNS validation**
6. In the **Key algorithm** section, I choose **RSA 2048**
7. In the Add tags sections, I create a tag name **"Website"** with value is my website **"www.nvo.one"** to keep track of my resource in AWS
8. Choose **Request** to be taken to the **Certificates** page.
9. Once the new certificate appears in **Pending** status, choose the certificate ID, and on the certificate details page, choose **Create record in Route53** to add the CNAME records to my domains, I used the same Route53 Hosted Zone above to create records in.
10. It took less than 30' to verify my domain, and the status will change to **Issued** when it's ready to be used.

# Step 4: Create and Set Up S3 bucket
I will need 2 buckets: one is for my subdomain **www.nvo.one** which will contains my static webside, and another one for my root **nvo.one** which will point to my subdomain bucket.

## Create and Set Up my sub-domain bucket
I need to create a bucket to store my static website content
1. Sign in to **AWS Management Console** and navigate to **S3** 
2. On the left panel, choose **Buckets**
3. Choose **Create bucket**
4. I name it **www.nvo.one**
5. Under **Block public access (bucket settings)**, untick **Block *all* public access**, choose **confirm** so that my bucket is accessible by public
6. Under **Bucket policy**, choose **Edit**, I copy the policy from AWS, modify my bucket name and save it
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::www.nvo.one/*"
        }
    ]
}
```
7. Under **Static website hosting**, I choose **Enable**
8. For **Hosting type**, I choose **Host a static website**
9. For **Index document**, I entered **"index.html"**
10. For **Error document**, I left it blank for now
11. Now the bucket is ready to use, I copy my website in
12. To test it, choose the bucket, on the nav menu, choose **Properties**, under **Static website hosting**, copy the bucket endpoints and paste it on the browser.

## Create and Set Up your root domain


Sure, here’s a rephrased version of that section:

To enable users to access my website without typing *www.*, I need to create an additional S3 bucket. This bucket will handle requests and redirect users to my main S3 bucket where the website content is hosted.

1. Create bucket
2. Name it **nvo.one** (without ***www.***)
3. Choose Properties
4. Under **Static website hosting**, choose **Edit**.
5. Choose **Redirect requests fr an object**.
6. In the **Host name** box, I entered my subdomain **nvo.one**.
7. For **Protocol**, choose **HTTPS**.
8. Choose **Save changes**

# Step 5: Create an Amazon CloudFront distribution for my subdomain www.nvo.one

I will need to also 2 CloudFront distribution: one for subdomain and one for root where they both using my SSL Certification.

## Subdomain distribution
1. Sign in to **AWS Management Console** and navigate to CloudFront.
2. Choose **Create Distribution**.
3. Under **Origin**, for **Origin domain**, I choose my Amazon S3 bucket that I created for my subdomain (ww.nvo.one).\
4. For the settings under **Default Cache Behavior Settings**, under **Viewer**, set **Viewer protocol policy** to **Redirect HTTP to HTTPS** and accept the default values for the rest.
5. For the fields under Settings, do the following:\
- Choose **Add item** for **Alternate domain (CNAME) - optional** and enter my subdomain name
- For **Custom SSL Certificate**, choose the certificate created above
- For **Default root object** text box, type in **index.html**
- I leave everything else default and choose **Create distribution**
6. Once the distribution created, I monitor the **Status** column, it will takes sometimes to change from **In progress** to **Deployed** where CloudFront will distribute my website to the world. How excited!

## Root domain distribution
I also want my root domain is distributed CDN to the world to server all the request to my root and of course the response will be redirected from my root bucket.

1. Sign in to **AWS Management Console** and navigate to CloudFront
2. Choose **Create Distribution**.
3. Under **Origin**, for **Origin Domain Name**, I enter my root S3 bucket endpoints.
4. For the settings under **Default Cache Behavior Settings**\
- Under **Viewer**, set **Viewer protocol policy** to **Redirect HTTP to HTTPS**
- Set **Cache settings** to **CachingDisabled**.
5. For the fields under Settings, do the following:\
- Choose **Add item** for **Alternate domain (CNAME) - optional** and enter my subdomain name
- For **Custom SSL Certificate**, choose the certificate created above
6. Choose **Create Distribution**
7. Give it sometimes, the **Status** will change from **In Progress** to **Deployed** that is when the distribution is ready to be used.

# Step 6: Route DNS traffic from my Domain name to my CloudFront distributions

As of now, I should have **a working domain name that is ready to be accessed by the world** and **2 CloudFront distributions which are ready to distribute my content (nvo.one and www.nvo.one)**. Next, I need to create a record for each of the distribution group to Route53 Hosted Zone where my domain name is using the same NS records.

## For subdomain
1. Sign in to **AWS Management Console** and navigate to Route53
2. On the left panel, choose **Hosted zones**
3. Under Hosted zones, choose my hosted zone **nvo.one**
4. Choose **Create record**.
5. Choose **Switch to wizard**.
6. Choose **Simple routing** and choose **Next**.
7. In **Record name**, I typed **www** in front of the default value
8. In **Record type**, choose **A - Routes traffic to an IPv4 address and some AWS resources**
9. in **Value/Route traffic to**, choose **Alias to CloudFront distribution**.
10. Choose my nvo.one CF distribution.
11. For **Evaluate target health**, I choose **No**.
12. Choose **Define simple record**.

## For root
1. Sign in to **AWS Management Console** and navigate to Route53
2. On the left panel, choose **Hosted zones**
3. Under Hosted zones, choose my hosted zone **www.nvo.one**
4. Choose **Create record**.
5. Choose **Switch to wizard**.
6. Choose **Simple routing** and choose **Next**.
7. In **Record name**, I typed **www** in front of the default value
8. In **Record type**, choose **A - Routes traffic to an IPv4 address and some AWS resources**
9. in **Value/Route traffic to**, choose **Alias to CloudFront distribution**.
10. Choose my nvo.one CF distribution.
11. For **Evaluate target health**, I choose **No**.
12. Choose **Define simple record**.

# Step 7: FAQ
## Invalidation
There are often a few times after updating my subdomain bucket, my website doesn't really update. Invalidation would help clear out the cache at AWS's edge location and force the edge location to pull from our S3 bucket again. This will guarantee user will see the updates immediately. But it will result a small charge which can accumulate over the time. 

Which will leads to another question that how can I know that my bucket already receive the update and will display it correctly before the edge location TTL runs out and sync up. By using S3 Static Web Hosting endpoint.

## S3 Static Web Hosting endpoint
Once the new update is copied to the S3 bucket, one way that we can see how to website looks like even before it will be pull to the CloudFront Edge's location is to use Statuc Web Hosting endpoint. 
1. Sign to **AWS Management Console**, navigate to **S3**
2. Choose to subdomain buckets (www.nvo.one)
3. Choose **Properties**, scroll to the end
4. Copy the **Static Web Hosting endpoints** and paste it into any browser

