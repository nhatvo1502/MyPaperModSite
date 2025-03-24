---
title: "Hosting a Python Flask Note Using AWS Services, Terraform, Github Actions"
date: 2025-03-07T07:54:28-08:00
draft: false
author: Nhat Vo
---

![image](/images/nnote/python-website.drawio.png)

During a conversation with a close friend in Australia, I was inspired by the idea of using a lightweight framework like Flask to build a cloud application. Thanks @terryduong for the inspiration and support for this project.

Check out my project on [GitHub](https://github.com/nhatvo1502/python-website)

# Stage 0: Game plan

I split this project into **five stages**.

1. Stage 1: Creating the app
2. Stage 2: Host locally
3. Stage 3: Hosting on AWS using Console
4. Stage 4: Hosting on AWS using Terraform
5. Stage 5: Automate the deployment using Terraform and Github Actions

A **Reference List** at the end of this page includes all the documents I consulted throughout this project.

# Stage 1: Creating Nnote

Thanks to Tech With Tim for this details [tutorial](https://www.youtube.com/watch?v=dam0GPOAvVI&t=407s&ab_channel=TechWithTim)

I used a **Python virtual environment** to track all dependencies via `requirements.txt`, facilitating the subsequent creation of a Docker image.

```bash
python -m venv .\venv
./env/scripts/activate

pip freeze > requirements.txt
```

\
After becoming familiar with Python Flask, I challenged myself to add some features to this app. A **password reset function** seemed useful.

\
First, I added a new button to the nav-bar in `base.html`

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

\
Then, I created a new page, `reset-password.html`

```html
<!--website/templates/reset-password.html-->
{% extends "base.html" %} {% block title %}Reset Password{% endblock %} {% block
content %}
<form method="POST">
  <h3 align="center">Reset Password</h3>
  <div class="form-group">
    <label for="password1">Current Password</label>
    <input
      type="password"
      class="form-control"
      id="current_password"
      name="current_password"
      placeholder="Enter current password"
    />
  </div>
  <div class="form-group">
    <label for="password2">New Password</label>
    <input
      type="password"
      class="form-control"
      id="new_password1"
      name="new_password1"
      placeholder="Enter new password"
    />
  </div>
  <div class="form-group">
    <label for="password2">New Password (Confirm)</label>
    <input
      type="password"
      class="form-control"
      id="new_password2"
      name="new_password2"
      placeholder="Re-enter new password"
    />
  </div>
  <br />
  <button type="submit" class="btn btn-primary">Submit</button>
</form>
{% endblock %}
```

\
Finally, I added a new route to `auth.py` to handle the logic.

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

# Stage 2: Hosting Locally

## 2a. Workflows:

1. Create DB
2. Run the app with Python
3. Integrate _Gunicorn_
4. Build app as Docker

**MySQL Server**
SQLite is lightweight and good for small project but AWS doesn't support this engine officially. I chose MySQL because it's cost-effective, user friendly and supported by AWS **[1]**.

Install [MySQL](https://dev.mysql.com/downloads/installer/) and [MySQL Workbench](https://dev.mysql.com/downloads/workbench/)

_Username:_ `root` \
_Password:_ `password` \
![image](/images/nnote/mysqlworkbench.png)

**Update Connection String and Database Authentication**
To have nnote-app interact with MySQL server, I used `sqlalchemy`

\
Install `sqlalchemy package`

```bash
pip install sqlalchemy_utils
```

\
Modified `__init__.py`

```python
#website/__init__.py
from sqlalchemy_utils import database_exists, create_database

#------------------------------------------------------------------

db = SQLAlchemy()
DB_USERNAME = "root"
DB_PASSWORD = "password"
DB_HOST = "host.docker.internal" # run from inside Docker
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

\
I will do a quick test to see if my **nnote-app** can interact with the DB

```bash
python .\main.py
```

... and voila, the **nnote-app** succesfully connect to MySQL Database on local and created 2 tables: `user` and `note` just as how I wanted.
![image](/images/nnote/localdb-test1.png)

Now, all we need to to pack all the dependencies into `requirements.txt` and get ready to build a Docker image. But before that, I need **Gunicorn**

**Gunicorn**

While Flask is great for development, its server is not suitable for concurrent traffics in production envrinonment. I choose Gunicorn **[2]** to handle Flask's shortcoming.

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

**Docker**
Install [Docker](https://docs.docker.com/desktop/setup/install/windows-install/)

I create `Dockerfile` under `.\` using `gunicorn` as my WGSI

```Dockerfile
FROM python:3.11

WORKDIR /app

COPY . .

RUN pip3 install -r requirements.txt

EXPOSE 5000

CMD ["gunicorn", "-b", "0.0.0.0:5000", "website:create_app()"]
```

## 2b. Testing:

build and run Docker

```bash
docker build -t nnote-app .
docker run -p 5000:5000 nnote-app
```

\
Docker Image is built succesfully, since I exposed `port 5000` from inside the Docker container to `port 5000` outside of the container, the correct URL to my nnote-app should be `localhost:5000` or `http://127.0.0.1:5000`
![image](/images/nnote/docker2.png)

**Stage 2 is completed!** I'm excited for the next one -- **AWS Cloud**.

# Stage 3: Hosting on AWS using Console

## 3a. Workflows:

What resources that I need on AWS: a MySQL Server, ECR->ECS->Task Definitaion->Service->Task(point back to my container image on ECR)

**Database**

1. Create a **MySQL db** by navigating to _AWS Console_ -> _Amazon RDS_ -> Select _Create database_
2. Configure:
   - Engine options = `MySQL`
   - Templates = `Free tier`
   - Master username = `admin`
   - Master password = `password`
   - Public access = `Yes`
3. Click _Create database_
4. Upon the database creation, I grab the `endpoint` from AWS RDS console and use **MySQL Workbench** test for connection and authentication.

![image](/images/nnote/aws-rds-1.png)

5. Update nnote-app with `endpoint`, `DB_USERNAME="admin"` and `DB_PASSWORD="password"`

```python
#website/__init__.py
#------------------------------------------------------------------

db = SQLAlchemy()
DB_USERNAME = "admin"
DB_PASSWORD = "password"
DB_HOST = "AWS-RDS-Endpoint" # run from inside Docker
DB_NAME = "nnote_database"

#------------------------------------------------------------------

def create():
	app = Flask(__name__)
	app.config['SECRET_KEY'] = 'mysecretkey'
	url = f"mysql+pymysql://{DB_USERNAME}:{DB_PASSWORD}@{DB_HOST}/{DB_NAME}"
	app.config['SQLALCHEMY_DATABASE_URI'] = url
```

**ECR**

1. To create a **Elastic Container Repo** **[3]**, navigate to _AWS Console_ -> _Amazon Elastic Container Registry_ -> _repository_ -> _Create depository_
2. Configure:
   - Repository name = `nnote-ecr/app`
   - Image tag mutability = `Mutable`
   - Encryption = `AES-256`
3. _Create_
4. Click our repo _nnote-ecr.app_ -> _View push commands_ -> follow the instruction from AWS to push my local Docker image to AWS ECR
   ![image](/images/nnote/ecr-push.png)
5. I also grabbed the the Repo URI on the way out for later

**ECS Cluster**

1. Navigate to _AWS Console_ -> _Amazon Elastic Container Service_ -> _Create cluster_
2. Configure:
   - Cluster name = `nnote-us-east-1-cluster`
   - Infrastructure = `AWS FarGate`
3. _Create_

**ECS Task definition**

1. In _AWS Elastic Container Service_ Console -> _Task Definition_
2. Configure - Infrastructure:
   - Task definition family = `nnote-td`
   - Launch Type = `AWS FarGate`
   - Task execution role = `ecsTaskExecutionRole`
3. Configure - Container - 1:
   - Name = `nnote-container`
   - Image URI = ${ERC's URI from above}:latest
   - Essential container = `Yes`
   - Container port = `80`

**ECS Cluster Service**

1. From _AWS Elastic Container Service_ console -> _Cluster_
2. Click on our cluster `nnote-us-east-1-cluster` -> under _Services_ tab -> _Create_
3. Configure:
   - Launch type = `FARGATE`
   - Application type = `Service`
   - Family = `nnote-td`
   - Service name = `nnote-service`
4. _Create_

If everything works as intented, our task will turn 1/1 shortly

![image](/images/nnote/ecs-service-1-wait.png)

Nothing good come easily. Something wrong with my image that cause my task to exit. Time for some troubleshootings.
![image](/images/nnote/ecs-service-1-error.png)
![image](/images/nnote/ecs-service-1-error-1.png)

## 3b. Debug:

I went back to my local computer to run `docker run` to see if my image is still working properly and receive this error
![image](/images/nnote/ecs-service-1-error-2.png)

\
Error `"Access denied for user 'root'@'74.212.237.134' (using password: YES)"` means db authentication isn't configured properly. I quickly check `__init__.py` and found my error. I rebuilt the container and run it locally. The error has gone.
![image](/images/nnote/ecs-service-1-error-fixed.png)

\
I push it to my ECR Repo and manually redeploy the Cluster service:

1. Navigate _AWS Elastic Container Service_ console -> Cluster
2. _nnote-us-east-1-cluster_ -> _Services_ -> _nnote-service_
3. Switch to _Tasks_ tab -> Under _Update service_ drop-down arrow > select _Force new deployment_

Finger cross...
![image](/images/nnote/ecs-service-2-wait.png)

Voila! The image seems to be interacting with the Database without any issue now.
![image](/images/nnote/ecs-service-2-success.png)

## 3c. Testing:

Let's test if nnote-app, first, we need the _container public IP address_

1. Amazon _Elastic Container Service_ -> _Clusters_ -> _nnote-us-east-1-cluster_ -> _Services_ -> _nnote-service_
2. Switch to _Tasks_ tab -> Select task -> switch to _Networking_
3. Copy Public Ip address
4. Paste on my browser and add `port 5000`

I can access my login screen!
![image](/images/nnote/stage2-complete-1.png)

\
Let's try sign-up...
![image](/images/nnote/stage2-complete-1-sign-up.png)

\
Then add some note...
![image](/images/nnote/stage2-complete-1-note.png)

\
Let's check our `nnote_database.user` on AWS RDS Database by navigate to MySQL Workbench > expand `nnote_database` > select `user` > enter this command `SELECT * FROM nnote_database.user`
![image](/images/nnote/stage2-complete-1-db-user.png)

\
How about `nnote_database.note`? `SELECT * FROM nnote_database.note`
![image](/images/nnote/stage2-complete-1-db-note.png)

Everything seems to be working as intented. But deploying this project involves a lot of manual work and the authentication between services are not secure. Let's streamline the CI/CD and enhance the authentication method at **Stage 4**

# Stage 4: Hosting on AWS using Terraform

I automate Stage 3 using Terraform but I still have to build and push Docker manually. I add my own VPC in `network.tf`.

```hcl
# terraform/network.tf
resource "aws_vpc" "nnote" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true

  tags = {
    Name = var.vpc_name
  }
}

resource "aws_internet_gateway" "nnote" {
  vpc_id = aws_vpc.nnote.id

  tags = {
    Name = "nnote_igw"
  }
}

resource "aws_subnet" "s1" {
  vpc_id            = aws_vpc.nnote.id
  cidr_block        = var.s1_cidr
  availability_zone = var.s1_az

  tags = {
    Name = "nnote_vpc_s1"
  }
}

resource "aws_subnet" "s2" {
  vpc_id            = aws_vpc.nnote.id
  cidr_block        = var.s2_cidr
  availability_zone = var.s2_az

  tags = {
    Name = "nnote_vpc_s2"
  }
}

resource "aws_route_table" "nnote" {
  vpc_id = aws_vpc.nnote.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.nnote.id
  }

  tags = {
    Name = "nnote_rt"
  }

  depends_on = [aws_internet_gateway.nnote]
}

resource "aws_route_table_association" "assoc1" {
  route_table_id = aws_route_table.nnote.id
  subnet_id      = aws_subnet.s1.id
}

resource "aws_route_table_association" "assoc2" {
  route_table_id = aws_route_table.nnote.id
  subnet_id      = aws_subnet.s2.id
}
```

\
and update `variables.tf`

```hcl
# terraform/variables.tf
# NETWORK
variable "vpc_name" {
  default = "nnote_vpc"
  type    = string
}

variable "vpc_cidr" {
  default = "10.0.0.0/16"
  type    = string
}

variable "s1_cidr" {
  default = "10.0.1.0/24"
  type    = string
}

variable "s1_az" {
  default = "us-east-1a"
  type    = string
}

variable "s2_cidr" {
  default = "10.0.2.0/24"
  type    = string
}

variable "s2_az" {
  default = "us-east-1b"
  type    = string
}

variable "vpc_sg_name" {
  default = "nnote_vpc_sg"
  type    = string
}
```

This stage went smoothly and I didn't encounter any issue which I was happy to cut it short to the **Final Stage**

# Stage 5: Automate deployment using Github Actions

The goal of this stage is to transform my app into a ready-to-deploy application that can be use by anyone

## 5a. Github Actions Workflow:

1. **Copy repo** to Github Actions Runner
2. **AWS OICD** to authenticate runner with AWS
3. **Create S3 bucket** to store Terraform backend tfstate file
4. **Deploy Terraform**
5. **Capture Terraform Output**
6. **Login AWS ECR**
7. **Build and Push Docker Image** to AWS ECR
8. **Force ECS Cluster Service's redeployment**
9. **Roll back if fail()** so that I don't have to clean up AWS Resources if something goes wrong, this is optional and I figure it would make my life so much better for testing

**AWS OICD**

I read **[4]** **[5]** to setup AWS OICD role. It seems very straight forward so I won't document these steps. You will need to configre this from your AWS IAM using your Github token.

\
**Injecting Secrets to Docker Image**

I configure `deploy.yml` to pick up `DB_NAME`, `DB_USERNAME`, and `DB_PASSWORD` from the Github Actions Secrets.

To setup **GHA Secrets**

1. Navigate to your Github repo
2. Select _Settings_ > _Secrets and variables_
3. Configure 3 variable with the exact name as below:
   - `DB_NAME`= _your-db-name_ (can be anything, this does not matter)
   - `DB_USERNAME` = _your-db-username_
   - `DB_PASSWORD` = _your-db-password_

\
**Injecting DB_HOST (RDS endpoints) to Docker Image**

Docker Image also needs the endpoints which is resulted from the Terraform's deployment. I output `endpoint` using `output.tf`, capture and exported it into Github Actions Environment.

```hcl
# terraform/output.tf
output "db_endpoint" {
  value = aws_db_instance.nnotedb.endpoint
}
```

I also output `ecr_uri`, `cluster_arn`, `service_name`, and `region` for login ECR, push Docker image and reploy cluster service later on.

```bash
# .github/workflows/deploy.yml
- name: Export Terraform output
        id: tf
        if: success()
        run: |
          echo "TF_EURI=$(terraform output -raw ecr_uri)" >> $GITHUB_ENV
          echo "TF_CARN=$(terraform output -raw cluster_arn)" >> $GITHUB_ENV
          echo "TF_SN=$(terraform output -raw service_name)" >> $GITHUB_ENV
          echo "TF_REGION=$(terraform output -raw region)" >> $GITHUB_ENV
          HOST=$(echo "$(terraform output -raw db_endpoint)" | cut -d ':' -f1)
          echo "TF_ENDPOINT=$HOST" >> $GITHUB_ENV
        working-directory: ./terraform
```

\
**Login AWS ECR**

```yml
- name: Log in to ECR
	run: aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $TF_EURI
```

**Build and Push Docker**
Along with **4c** and **4d**, I inject these 4 variables while building the Docker image

```yml
- name: Build Docker
	run: |
		docker build \
		--build-arg DB_USERNAME=${{ secrets.DB_USERNAME }} \
		--build-arg DB_PASSWORD=${{ secrets.DB_PASSWORD }} \
		--build-arg DB_NAME=${{ secrets.DB_NAME }} \
		--build-arg DB_HOST=${{ env.TF_ENDPOINT }} \
		-t nnote-app .
- name: Tag Docker
	run: docker tag nnote-app:latest $TF_EURI:latest

- name: Push Docker
	run: docker push $TF_EURI:latest
```

**Redeploy Cluster Service**

```yml
- name: Force Redeployment ECS
	run: aws ecs update-service --cluster $TF_CARN --service $TF_SN --force-new-deployment --region $TF_REGION
```

**Roll back if fail()**

```yml
- name: Auto clean up if Apply failed
	if: failure()
	run: |
		terraform destroy --auto-approve -var='db_name="a"' -var='db_password="b"' -var='db_username="c"'
		aws s3 rm s3://nnote-tfstate-031225 --recursive
		aws s3 rb s3://nnote-tfstate-031225 --force
	working-directory: ./terraform
```

## 5b. Debugging:

I ran into a couple of issues

**Issue 1:** RDS Endpoint from Terraform output contains `:3306`
Solution: cut the `:3306` from the string

```yml
# .github/workflows/deploy.yml
HOST=$(echo "$(terraform output -raw db_endpoint)" | cut -d ':' -f1)
echo "TF_ENDPOINT=$HOST" >> $GITHUB_ENV
```

**Issue 2:** Container cannot authenticate with Database unless I use MySQL Workbench to drop the schemas and redeploy Cluster Service
Solution: remove `db_name = var.db_name` when create aws_rds_instance

Before...

```hcl
resource "aws_db_instance" "nnotedb" {
  db_name                = var.db_name
  allocated_storage      = 20
  engine                 = "mysql"
  engine_version         = "8.0"
  instance_class         = var.db_instance_class
  username               = var.db_username
  password               = var.db_password
  vpc_security_group_ids = [aws_security_group.nnote_vpc_sg.id]
  db_subnet_group_name   = aws_db_subnet_group.nnotedb.name
  skip_final_snapshot    = true
  publicly_accessible    = true
}
```

After...

```hcl
resource "aws_db_instance" "nnotedb" {
  allocated_storage      = 20
  engine                 = "mysql"
  engine_version         = "8.0"
  instance_class         = var.db_instance_class
  username               = var.db_username
  password               = var.db_password
  vpc_security_group_ids = [aws_security_group.nnote_vpc_sg.id]
  db_subnet_group_name   = aws_db_subnet_group.nnotedb.name
  skip_final_snapshot    = true
  publicly_accessible    = true
}
```

# Reference List:

**[1]** https://www.greengeeks.com/blog/sqlite-vs-mysql/#:~:text=Ultimately%2C%20SQLite%20is%20a%20lightweight,go%2Dto%20for%20RDBMS%20solutions.

**[2]** https://flask.palletsprojects.com/en/stable/deploying/gunicorn/

**[3]** https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-container-image.html

**[4]** https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html

**[5]** https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services
