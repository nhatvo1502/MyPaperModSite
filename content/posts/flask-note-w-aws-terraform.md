---
title: "Hosting A Python Flask Note using AWS Services, Terraform, Github Action"
date: 2025-03-07T07:54:28-08:00
draft: true
author: Nhat Vo
---

During one of the conversation with a closed friend in Australia, I was inspired by Flask and the idea of using a lightweight webframework to build a web application. Thanks @terryduong for the inspiration and support through out this project.

# Stage 0: Game plan
The goal is too create a web application that allow my users to create login and take. I will host this app completely on AWS Cloud. This project has **6 Stages**. 
1. Stage 1: Creating the app
2. Stage 2: Host locally
3. Stage 3: Publish the app on AWS using AWS Console
4. Stage 4: Automate the deployment using Terraform and Github Action
5. Stage 5: Testing/Debugging
6. Stage 6: Documentation and blogging

There is a **Reference List** in the end of this page includes all the documents I read to through out this project.

# Stage 1: Creating nnote

Thanks to Tech With Tim for this details [tutorial](https://www.youtube.com/watch?v=dam0GPOAvVI&t=407s&ab_channel=TechWithTim)

## a. Python Virtual Environment

I will need to build a docker image for hosting by the end of Stage 2, I used Python Virtual Envinronment to have all of the dependencies documented along the way.

Create virtual env
```bash
python -m venv .\venv
```

Activate virtual env
```bash
./env/scripts/activate
```

I made a few customizations to the app:

## b. Add New Feature

After getting familiar with Python Flask, I challenged myself to add a new function. I feel like **Reset Password** can be really appreciated here.

\
add this code-block to `auth.py`
```python
#website/auth.py
@auth.route('/reset-password', methods=['GET', 'POST'])
def reset_password():
	if request.method == 'POST':
		current_password = request.form.get('current_password')
		new_password1 = request.form.get('new_password1')
		new_password2 = request.form.get('new_password2')

		if check_password_hash(current_user.password, current_password) == False:
			flash('Incorrect password!', category='error')
		else:
			if len(new_password1) < 7:
				flash('Password must be at least 7 characters.', category='error')
			elif new_password1!=new_password2:
				flash('Password don\'t match', category='error')
			else:
				current_user.password = generate_password_hash(new_password1, method='pbkdf2:sha256')
				db.session.commit()
				flash('Password  reset succesfully! Please login again.')
				logout_user()
				return redirect(url_for('auth.login'))
	return render_template('reset_password.html', user=current_user)
```
\
add to nav-bar in `base.html` 
```html
<!--website/templates/base.html-->
<div class="collapse navbar-collapse" id="navbar">
  <div class="navbar-nav">
    {% if user.is_authenticated %}
    <a class="nav-item nav-link" id="home" href="/">Home</a>
    <a class="nav-item nav-link" id="resetPassword" href="/reset-password"
      >Reset Password</a
    >
    <a class="nav-item nav-link" id="logout" href="/logout">Logout</a>
    {% else %}
    <a class="nav-item nav-link" id="login" href="/login">Login</a>
    <a class="nav-item nav-link" id="signUp" href="/sign-up">Sign Up</a>
    {% endif %}
  </div>
</div>
```

# Stage 2: Hosting Locally

## a. Create MySQL Server:
Tim used SQLite in his tutorial because it's lightweight and good for small project but it's not supported by AWS. I need a different database engine that is supported by AWS, cost-effective and friendly. For these reason, I choose MySQL **[1]**.

Install [MySQL](https://dev.mysql.com/downloads/installer/) and [MySQL Workbench](https://dev.mysql.com/downloads/workbench/)

*Username:* `root` \
*Password:* `password` \
![image](/images/nnote/mysqlworkbench.png)

## b. Update Connection String and Database Authentication:
From the virtual environment, install sqlalchemy_utils
```bash
pip install sqlalchemy_utils
```
Modify `__init__.py`

```python
#website/__init__.py
from sqlalchemy_utils import database_exists, create_database

#------------------------------------------------------------------

db = SQLAlchemy()
DB_USERNAME = "root"
DB_PASSWORD = "password"
DB_HOST = "host.docker.internal" # run from inside docker
DB_NAME = "nnote_database"

#------------------------------------------------------------------

def create():
	app = Flask(__name__)
	app.config['SECRET_KEY'] = 'mysecretkey'
	url = f"mysql+pymysql://{DB_USERNAME}:{DB_PASSWORD}@{DB_HOST}/{DB_NAME}"
	app.config['SQLALCHEMY_DATABASE_URI'] = url

#------------------------------------------------------------------

def create_db(url, app):
    # Check if the database exists, if not, create it
    print('create_database is running')
    if not database_exists(url):
        create_database(url)
        # Use the app context to create all tables
        with app.app_context():
            db.create_all()
            print('Created Database!')
```

## c. Quick test:
I ran my project ...
```bash
python .\main.py
```
... and voila, the *nnote* succesfully connect to MySQL Database on local and created 2 tables: `user` and `note` just as how I wanted.
![image](/images/nnote/localdb-test1.png)

Now, all we need to to pack all the dependencies into `requirements.txt` and get ready to build a docker image. But before that, we have one issue...

## d. Gunicorn

While Flask is great for development, its server is not suitable for concurrent traffics, which we need after moving the project to AWS. I choose Gunicorn **[2]** to handle this issue.

\
To install...
```bash
pip install gunicorn
```

\
export dependencies...
```bash
pip freeze > requirements.txt
```

## e. Install Docker
Install [Docker](https://docs.docker.com/desktop/setup/install/windows-install/)

## f. Dockerfile

I create `Dockerfile` under `.\` using `gunicorn` as my WGSI
```Dockerfile
FROM python:3.11

WORKDIR /app

COPY . .

RUN pip3 install -r requirements.txt

EXPOSE 5000

CMD ["gunicorn", "-b", "0.0.0.0:5000", "website:create_app()"]
```
\
\
*build and run docker*
```bash
docker build -t flask-app .
docker run -p 5000:5000 flask-app
```
\
Docker Image is built succesfully, since I exposed `port 5000` from inside the docker container to `port 5000` outside of the container, the correct URL to my nnote-app should be `localhost:5000` or  `http://127.0.0.1:5000`
![image](/images/nnote/docker2.png)

Stage 2 is completed, let's get to the most exciting part of this project - **AWS Cloud**.

# Stage 3: Hosting on AWS

## a. RDS
1. Navigate to *AWS Console* -> *Amazon RDS* -> Select *Create database*
2. Configure:
	- Engine options = `MySQL`
	- Templates = `Free tier`
	- Master username = `admin`
	- Master password = `password`
	- Public access = `Yes`
3. Click *Create database*

Upon the database creation, I grab the endpoint and use MySQL Workbench to connect just to test the connection.

![image](/images/nnote/aws-rds-1.png)

## b. Update nnote with AWS RDS Connection String and Authentication
```python
#website/__init__.py
#------------------------------------------------------------------

db = SQLAlchemy()
DB_USERNAME = "admin"
DB_PASSWORD = "password"
DB_HOST = "AWS-RDS-Endpoint" # run from inside docker
DB_NAME = "nnote_database"

#------------------------------------------------------------------

def create():
	app = Flask(__name__)
	app.config['SECRET_KEY'] = 'mysecretkey'
	url = f"mysql+pymysql://{DB_USERNAME}:{DB_PASSWORD}@{DB_HOST}/{DB_NAME}"
	app.config['SQLALCHEMY_DATABASE_URI'] = url
```

## c. ECR
To host docker on AWS, I used ECS **[3]***
1. Navigate to *AWS Console* -> *Amazon Elastic Container Registry* -> *repository* -> *Create depository*
2. Configure:
	- Repository name = `nnote-ecr/app`
	- Image tag mutability = `Mutable`
	- Encryption = `AES-256`
3. *Create*
4. Click our repo *nnote-ecr.app* -> *View push commands* -> follow the instruction from AWS to push my local docker image to AWS ECR
![image](/images/nnote/ecr-push.png)
5. I also grabbed the the Repo URI on the way out for later

## d. ECS Cluster
1. Navigate to *AWS Console* -> *Amazon Elastic Container Service* -> *Create cluster*
2. Configure:
	- Cluster name = `nnote-us-east-1-cluster`
	- Infrastructure = `AWS FarGate`
3. *Create*

## e. ECS Task definition
1. In *AWS Elastic Container Service* Console -> *Task Definition*
2. Configure - Infrastructure:
	- Task definition family = `nnote-td`
	- Launch Type = `AWS FarGate`
	- Task execution role = `ecsTaskExecutionRole`
3. Configure - Container - 1:
	- Name = `nnote-container`
	- Image URI = ${ERC's URI from above}:latest
	- Essential container = `Yes`
	- Container port = `80`

## f. ECS Cluster Service
1. From *AWS Elastic Container Service* console -> *Cluster* 
2. Click on our cluster `nnote-us-east-1-cluster` -> under *Services* tab -> *Create*
3. Configure:
	- Launch type = `FARGATE`
	- Application type = `Service`
	- Family = `nnote-td`
	- Service name = `nnote-service`
4. *Create*

If everything works as intented, our task will turn 1/1 shortly

![image](/images/nnote/ecs-service-1-wait.png)

Nothing good come easily. Something wrong with my image that cause my task to exit 
![image](/images/nnote/ecs-service-1-error.png)
![image](/images/nnote/ecs-service-1-error-1.png)

## g. Troubleshooting
I went back to my local computer to run `docker run` to see if my image is still working properly and receive this error
![image](/images/nnote/ecs-service-1-error-2.png)

Error "Access denied for user 'root'@'74.212.237.134' (using password: YES)" which mean my username and password to AWS RDS Database are incorrected. I check the DB credential my `__init__.py` and found my error. I build and succesfully run `docker run` again
![image](/images/nnote/ecs-service-1-error-fixed.png)

I rebuild my docker image and push it to my ECR Repo. AWS ECS Cluster Service won't restart itself automatically, I need to manually redeploy it:
1. Navigate *AWS Elastic Container Service* console -> Cluster
2. *nnote-us-east-1-cluster* -> *Services* -> *nnote-service*
3. Switch to *Tasks* tab -> Under *Update service* drop-down arrow select *Force new deployment*

Finger cross...
![image](/images/nnote/ecs-service-2-wait.png)

Voila!
![image](/images/nnote/ecs-service-2-success.png)

## h. Testing
Let's test if nnote is working!
1. Amazon *Elastic Container Service* -> *Clusters* -> *nnote-us-east-1-cluster* -> *Services* -> *nnote-service*
2. Switch to *Tasks* tab -> Select task -> switch to *Networking*
3. Copy Public Ip address
4. Paste on my browser and add `port 5000`

I can access my login screen!
![image](/images/nnote/stage2-complete-1.png)

Let's try sign-up...
![image](/images/nnote/stage2-complete-1-sign-up.png)

Then add some note...
![image](/images/nnote/stage2-complete-1-note.png)

Let's check our `user table` on AWS RDS Database
![image](/images/nnote/stage2-complete-1-db-user.png)

How about `note table`?
![image](/images/nnote/stage2-complete-1-db-note.png)

Everything seems to be working as intented. But deploying this project involves a lot of manual work and the authentication between services are not secure. Let's move on the next stage


# Stage 4: Automate the deployment using Terraform and Github Action

## a. CI/CD Pipeline:
I will streamline my project's deployment and updating by leveraging Github Action and Terraform. In order for my Github Action to communicate with AWS Services, I need to setup an authentication. I had setup AWS Credential using Github Action Environment Secrets before but I will learn and use AWS OICD  for this project.

## b. AWS OICD:
I read **[4]** **[5]** to setup AWS OICD role

## c. GHAction deploy workflow
`terraform\main.tf`
Clone Repo to the Runner -> Setup OICD -> Create tfstate backend bucket -> Install and Deploy Terraform -> Export Terraform Outputs -> Build and Push Docker Image -> Force ECS Re-deployement

## d. Passing DB_USERNAME and DB_PASSWORD


## d. Passing variables from Terraform Output -> Docker Image
Since all of the infrastructure instances will be created after the code has been pushed, I used Terraform output to export these values and GitHub Environment variable to capture.
`__init__.py`

## e. GHAction Environment Secrets

## e. Troubleshooting
## f. Testing

# Stage 5: Testing/Debugging

# Stage 6: Documentation and bloggin

# Reference List:

**[1]** https://www.greengeeks.com/blog/sqlite-vs-mysql/#:~:text=Ultimately%2C%20SQLite%20is%20a%20lightweight,go%2Dto%20for%20RDBMS%20solutions.

**[2]** https://flask.palletsprojects.com/en/stable/deploying/gunicorn/

**[3]** https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-container-image.html

**[4]** https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html

**[5]** https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services

