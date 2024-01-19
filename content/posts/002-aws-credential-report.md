---
title: 'Using Github Action to generate AWS Credential Report'
date: 2024-01-15T10:10:29-08:00
draft: false
categories:
    - AWS
    - GitHub Action
---

## I. Introduction
This project is inspired by one of my dear friend. Instead of using the AWS Console to generate a credential report, we would set up a runner on GitHub Actions to automate this task. It's a very cool experiment, and I hope that by the end, I can apply and scale this concept into a part of a bigger project down the road.

## II. Our Game-plan
![image](/images/aws-credential-report/game-plan.png)
We will leverage Github Action to execute a set of commands to generate AWS Credential Report, download it, decode, pipe the content into csv file and finally copy to a existing S3 bucket.

Tools that we need for this project:
1. AWS CLI
```
https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
```
2. GitHub CLI
```
https://cli.github.com/
```

## III. Create an AWS CLI user + proper permission
Steps: 
1. Create custom IAM policy on AWS Console
2. Create CLI user
3. Attach the custom IAM Policy to CLI user
4. Create a CLI Access Key
5. Confiure AWS Credential on local terminal

*What permission do you need?*\
*- Generate Credential Report **iam:GenerateCredentialReport***\
*- Download Credential Report **iam:GetCredentialReport***\
*- Write to S3 **S3:PutObject***\
*- List S3 bucket **S3:GetPbject***

***1. Create custom IAM policy on AWS Console:***\
 AWS Console > IAM > Under Access management > Policies
![image](/images/aws-credential-report/AWS_Permission_group.png)

Copy and paste the following JSON policy to the policy editor. You can also create a seperate policy for S3:PutObject where it target one of your bucket.
```
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "VisualEditor0",
			"Effect": "Allow",
			"Action": [
                          "iam:GenerateCredentialReport",
                          "s3:PutObject",
                          "iam:GetCredentialReport",
                          "s3:GetObject"
			],
			"Resource": "*"
		}
	]
}
```
***2. Create CLI user***
Create a CLI user via AWS Console > IAM > Under Access management > Users > Create user
![image](/images/aws-credential-report/aws-create-user.png)

***3. Attach the custom IAM Policy to CLI user***
Select CLI User > Under Permissions > Permissions policies > Add permission > Select our custom policies
![image](/images/aws-credential-report/policy.png)

***4. Create a CLI Access Key***
Select CLI User in AWS Console > Security credentials Under Access keys > Create access key > Select Command Line Interface (CLI) > tick the box Confirmation > Next > Create access key\
Note down the Access key and Secret access key, you will need these later
![image](/images/aws-credential-report/accesskey.png)

***5. Confiure AWS Credential on local terminal***\
Go to local terminal or Ctrl + Shift +` in vscode to open terminal (my default terminal is PowerShell) > enter this command
```
aws configure
```

- Copy and paste your CLI user access key

- Copy and paste your CLI user secret access key

- Enter default region (mine is us-west-1)

- Enter default format (mine is json)

- Verify your input by entering this command

```
aws configure list
```
![image](/images/aws-credential-report/aws-configure-list.png)


## IV. GitHub

Steps:
1. Create Github account from their website if you don't have one ready
2. Create a repository
3. Create secret environment
4. Create AWS access key and secret access key secrets

***1. Create Github account from their website if you don't have one ready***

Go to their website and sign-up
```
https://github.com/
```
***2. Create a repository***

You can create a new repo from Github web console or using CLI:
Login
```
gh auth login
```
Create new repo
```
gh repo create
```
If you create repo from web, make sure to clone it to your local
```
gh repo clone
```

***3. Create Github Action AWS Secrets***

Go to github.com > Your project repository > Settings > under Security > Secrets and variables > Actions

New Environment > Create environment name Dev > Click on Dev

Add secret > Name it ***AWS_ACCESS_KEY_ID*** > Copy and paste your CLI Access Key from the note

Add secret > Name it ***AWS_SECRET_ACCESS_KEY*** > Copy and paste your CLI Secret Access Key from the note

![image](/images/aws-credential-report/gh-action-secrets.png)

## V. VSCODE extension**

I use VSCode along with these extensions ![image](/images/aws-credential-report/github-action.png)

**GitHub Action** will support syntax for yml file.

## VI. Github Action

Steps:
1. Create .github/workflows
2. Create yaml files
3. Configure yaml files
4. Code explains
5. Commit to Github
6. Test

***1. Create .github/workflows***

Go to the repo folder and create this folder structure
```
mkdir .github
cd .github
mkdir workflows
```
Create action.yml\
It should look like this\
![image](/images/aws-credential-report/folder-structure.png)


***3. Configure yaml files***

Open action.yml using vscode

Copy and paste this code block
```
name: Automate Credential Report
on:
  # trigger manually
  workflow_dispatch:

jobs:
  generate-export-credential-report:
    runs-on: ubuntu-latest
    environment: Dev

    steps:
      - name: Verify aws
        run: aws --version

      - name: Configure AWS Credentials
        env: 
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_DEFAULT_REGION: 'us-east-1'
        run: |
          mkdir -p ~/.aws
          echo "[default]" > ~/.aws/credentials
          echo "aws_access_key_id=${AWS_ACCESS_KEY_ID}" >> ~/.aws/credentials
          echo "aws_secret_access_key=${AWS_SECRET_ACCESS_KEY}" >> ~/.aws/credentials
          echo "region=${AWS_DEFAULT_REGION}" >> ~/.aws/credentials
      
      - name: Verify AWS Credential
        run: aws configure list

      - name: Verify s3 permission
        run: aws s3 ls
      
      - name: Generate Credential Report
        run: aws iam generate-credential-report
      
      - name: Wait for 5 second
        run: sleep 5
      
      - name: Retrieve report
        run: report_content=$(aws iam get-credential-report --query 'Content' --output text)
      
      - name: Decode report
        run: report_text=$(echo "$report_content" | base64 -d)
      
      - name: Save report to csv file
        run: echo "$report_text" > credential-report.csv
      
      - name: Copy the report to s3
        run: aws s3 cp ./credential-report.csv s3://keypair-1234
```

***4. Code explains***

- We will start with giving this action a name
```
name: Automate Credential Report
```

- The ***on*** code block will determine the trigger condition, in this case, ***work_dispatch*** mean manual trigger. Read more on on: 
```
https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions
```

```
on:
    workflow_dispatch
```


- The jobs structure should be
```
jobs:
    build:
        runs-on: *The machine that you want the jobs to be ran on, we call it runner*
        environment: *Environment to store our secret on git hub*

        steps:
            - name: *Name of each step
              run: *bash command*
```
- Using this understanding, it will translate our code above into
```
jobs:
  generate-export-credential-report: *our build name*
    runs-on: ubuntu-latest *running on abuntu with latest update*
    environment: Dev *Our Github secret environment Dev*
```

- All ubuntu-latest on github action pre-installed with AWS CLI, we just want to verify if this is true by verify aws version
```
- name: Verify aws
        run: aws --version
```

- Login AWS CLI user to this runner using AWS Access Key and Secret Access Key that we have saved into our note above. Instead of writing our secret key to the yml, we will be passing it from our Github secret environment which I will be showing in a bit.
```
- name: Configure AWS Credentials
        env: 
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_DEFAULT_REGION: 'us-east-1'
        run: |
          mkdir -p ~/.aws
          echo "[default]" > ~/.aws/credentials
          echo "aws_access_key_id=${AWS_ACCESS_KEY_ID}" >> ~/.aws/credentials
          echo "aws_secret_access_key=${AWS_SECRET_ACCESS_KEY}" >> ~/.aws/credentials
          echo "region=${AWS_DEFAULT_REGION}" >> ~/.aws/credentials
```

- Verify our credential 
```
- name: Verify AWS Credential
        run: aws configure list
```
- This step is optional since I only include S3:PutObject in the policy. To be able to verify if we have permision to s3, we also need S3:GetObject.
```
- name: Verify s3 permission
        run: aws s3 ls
```
- Generate AWS Credential Report
```
- name: Generate Credential Report
  run: aws iam generate-credential-report
```
- Sleep for 5 to make sure the AWS finish generate the report before we can download, if we try to download right after generate, there is a chance the report has not yet been created therefore the script will timeout
```
- name: Wait for 5 second
  run: sleep 5
```

- Download the report into a variable ***report_content***
```
- name: Retrieve report
  run: report_content=$(aws iam get-credential-report --query 'Content' --output text)
```

- Since the content we just downloaded is encrypted Base64, we need to decode it
```
- name: Decode report
  run: report_text=$(echo "$report_content" | base64 -d)
```

- Save it as csv file
```
- name: Save report to csv file
  run: echo "$report_text" > credential-report.csv
```

- Then copy it to S3
```
- name: Copy the report to s3
  run: aws s3 cp ./credential-report.csv s3://keypair-1234
```

***5. Commit to Github***

Enter this command to commit the code to github
```
git add *.*
git commit -m 'my first commit'
git push
```
Go to github.com > your repo and check if your code is there

Go to Actions tab > if the folder structure .github/workflows/action.yml was done right, you should see this
![image](/images/aws-credential-report/gh-action1.png)

Drop down Run workflow button > Run workflow > Click onto the workflow and see all the steps are executing in real time
![image](/images/aws-credential-report/gh-action2.png)

If all the steps are succesfully executed > go to AWS Console > S3 > your bucket > you should see your report
![image](/images/aws-credential-report/report-s3.png)

Download it and open with excel to see if the decoding works
