---
title: 'How I host my website on AWS using Domain from Godaddy.com'
date: 2024-01-29T11:16:33-08:00
draft: false
author: Nhat Vo
---

# Intro
![image](/images/007/Drawing8.png)

# Step 1: Purchase my first domain
At first, I planned to buy my first domain from AWS but they don't support *.one* so I bought from GoDaddy.com.

# Step 2: Update Domain Nameserver with AWS Route53 generated NS

Since the domain that I bought from Godaddy.com is currently using the nameservers from Godaddy.com. I would need to update these NS, so that any traffic to my domain **nvo.one** will be routed to my AWS CloudFront.

## Generate AWS NS Records
1. Sign in to **AWS Management Console** and navigate to Route53.
2. On the left panel, select **Hosted zones**.
3. Choose **Create hosted zone**.
4. Under **Domain name**, I entered nvo.one
5. Under **Type**, I left it unchanged as **Public hosted zone**.
6. Under **Tags**, choose **Add tag**, with **Key** is **"Website"** and **Value** is **"www.nvo.one"*** to keep track of my resources later on.
7. Choose **Create Hosted Zone**
8. Route53 will automatically create a new set of NS with 4 records, I left this tab open and will come back to copy these value for the next step.

## Update my domain NS Record
1. Sign in to Godaddy.com console
2. Choose the "Nine dots" next to my Profile to expand the nav menu.
3. Choose **Domains**
4. Highlight my domain name, choose the "3 dots" to expand the settings bar and choose **Edit DNS**
5. Under my domain **Domain Settings* menu bar, choose **Nameservers**
6. Choose **Change Nameservers**
7. Godaddy asks for my confirmation by replacing the Nameservers record, it will remove Godaddy NS, choose **Yes**
8. I go back to my AWS Route53 from the previous tab to copy each NS Records and paste in GoDaddy console. By default,GoDaddy only provide 2 NS records. I increase these number of records to 4 to match with what AWS Route53 provides.

# Step 3: Request a public certificate

My website will requires a public certificate in order for Amazon CloudFront distributions to configure CloudFront to require viewers use HTTPS so that connections are encrypted when CoudFront communicate with viewers.

CloudFront can only use certificate created in US East (N. Virginia) Region. If you create a certificate and later set up CloudFront and your CloudFront couldn't find it, there is a good change your certificate was created in a region which is not US East. Forunately, AWS Certification is free and easy to create. Here is how I create mine:

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

I also want my users to be able to browse to my website without typing *www.*, I need to create another bucket to response to this request and redirect the users to my S3 bucket where the content is at.
1. Create bucket
2. Name it **nvo.one** (without ***www.***)
3. Choose Properties
4. Under **Static website hosting**, choose **Edit**.
5. Choose **Redirect requests fr an object**.
6. In the **Host name** box, I entered my subdomain **nvo.one**.
7. For **Protocol**, choose **HTTPS**.
8. Choose **Save changes**

# Step 5: Create an Amazon CloudFront distribution for my subdomain www.nvo.one

CloudFront is a Content Delivery Network service from AWS (CDN) that I use to improve traffic speed from user to my website and also enable HTTPS so people can view it securely. I would need to create one for subdomain S3 bucket and one for my root S3 bucket

## For subdomain
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

## For root
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

Up to this point, I have a working domain name that is ready to be accessed by the world and 2 CloudFront distributions which are ready to distribute my content (nvo.one and www.nvo.one). I need to "connect" (route the traffic) together via Route53.

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


