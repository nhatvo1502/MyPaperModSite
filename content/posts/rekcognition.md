---

title: 'Rekognition'
date: 2025-02-27T13:35:59-08:00
draft: false
author: Nhat Vo
---------------
![image](/images/rekognition/Rekognition.drawio.png#center)

## Intro:
Recently, I explored a simple yet powerful way to achieve image processing using AWS Lambda and AWS Rekognition. With Machine Learning on the rise, I think it would be a cool project for me to get my hands on this topic.
![image](/images/rekognition/About_WildBird-mobile.jpg#center)

![image](/images/rekognition/meta2.png#center)

In this post, I’ll walk through a scalable and cost-effective approach: using AWS Lambda to process images stored in an S3 bucket, analyze their content with Rekognition to identify objects, faces, or text, and then store the extracted metadata in another S3 bucket for further use.

## Terraform:
### 1. S3
I created two S3 buckets: one for input images and one for storing Rekognition metadata and resized images. While it is possible to use a single bucket, I would not recommend it, as putting a new image into the same bucket that triggers the Lambda function could cause the application to enter an infinite loop.
```terraform
# S3
resource "aws_s3_bucket" "input_bucket" {
  bucket = "serverless-image-input-bucket"
}

resource "aws_s3_bucket" "output_bucket" {
  bucket = "serverless-image-out-bucket"
}
```
### 2. Lambda
I created a Lambda function using Python 3.9 and used a customized layer from the user "Klayers" to import the PIL library.

```terraform
# Lambda Function
resource "aws_lambda_function" "image_processor" {
  filename      = "lambda_function.zip"
  function_name = "image-processing-lambda"
  role          = aws_iam_role.lambda_role.arn
  handler       = "lambda_function.lambda_handler"
  runtime       = "python3.9"
  timeout       = 10

  source_code_hash = filebase64sha256("lambda_function.zip")

  layers = ["arn:aws:lambda:us-east-1:770693421928:layer:Klayers-p39-pillow:1"]

  environment {
    variables = {
      OUTPUT_BUCKET = aws_s3_bucket.output_bucket.id
    }
  }
}

resource "aws_lambda_permission" "allow_s3" {
  statement_id  = "AllowS3Invoke"
  action        = "lambda:InvokeFunction"
  principal     = "s3.amazonaws.com"
  function_name = aws_lambda_function.image_processor.function_name
  source_arn    = aws_s3_bucket.input_bucket.arn
}
```

### 3. Permissions and Roles

I created a custom inline IAM role for the Lambda function and granted it the necessary permissions to execute tasks involving both S3 buckets and the CloudWatch LogGroup.

```terraform
resource "aws_iam_role" "lambda_role" {
  name = "lambda-image-processing-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "lambda.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_policy_attachment" "lambda_policy" {
  name       = "lambda-s3-rekognition-policy"
  roles      = [aws_iam_role.lambda_role.name]
  policy_arn = "arn:aws:iam::aws:policy/AmazonS3FullAccess"
}

resource "aws_iam_policy_attachment" "lambda_rekognition_policy" {
  name       = "lambda-rekognition-policy"
  roles      = [aws_iam_role.lambda_role.name]
  policy_arn = "arn:aws:iam::aws:policy/AmazonRekognitionFullAccess"
}

resource "aws_iam_role_policy" "logging" {
  name = "lambda-image-processing-logging-policy"
  role = aws_iam_role.lambda_role.name

  policy = jsonencode({
    Version = "2012-10-17",
    Statement = [{
      Effect   = "Allow",
      Action   = [
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      Resource = ["arn:aws:logs:*:566027688242:log-group:*:log-stream:*"]
    }]
  })
}
```

### 4. S3 Triggers

Finally, I created a trigger event so that every time a new object is added to the input S3 bucket, it will invoke the Lambda function to process the image. The `depends_on` attribute ensures that the Lambda function is created before the trigger is assigned.

```terraform
resource "aws_s3_bucket_notification" "s3_trigger" {
  bucket = aws_s3_bucket.input_bucket.id

  lambda_function {
    lambda_function_arn = aws_lambda_function.image_processor.arn
    events              = ["s3:ObjectCreated:*"]
  }

  depends_on = [aws_lambda_function.image_processor] # Ensure the lambda function is created first
}
```

## Deploy:

### 1. AWS CLI

I created an AWS CLI account with full permissions since I will deploy the Terraform files remotely from my local computer.

### 2. Terraform CLI:

```sh
terraform fmt # Format main.tf to ensure consistency in syntax
terraform validate # Validate the configuration
terraform plan # Review what I'm about to deploy
terraform apply --auto-approve # Deploy main.tf
```

## Testing:

The test is straightforward. I upload a picture of a random animal into the input S3 bucket. The presence of a new object in this bucket triggers the Lambda function, which also logs an entry in the CloudWatch LogGroup. The Lambda function performs two tasks:
1. Makes a copy of the original picture, resizes it, and stores it in the output bucket.
2. Uses AWS Rekognition to analyze the image content and outputs the metadata as a JSON file into the output bucket.

But first thing first, I will check if my terraform file has been deployed correctly by running this command:

```sh
terraform state list
```
![image](/images/rekognition/terraformstatelist.png#center)

It works perfectly, and here is the result.
![image](/images/rekognition/s3output.png#center)

## Refinement:

I have only scratched the surface of AWS Rekognition's use cases. This small application can be scaled up to handle a large volume of images in bulk where Rekognition has a lot more to offer.

## Resources and Credits
*Klayer https://github.com/keithrozario/Klayers* \
*Rekognition Python https://docs.aws.amazon.com/rekognition/latest/customlabels-dg/ex-lambda.html*

Check out this project on my [GitHub](https://github.com/nhatvo1502/aws-rekognition-image-processing).