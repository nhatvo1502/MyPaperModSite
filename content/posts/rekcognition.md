---

title: 'Rekognition'
date: 2025-02-27T13:35:59-08:00
draft: false
author: Nhat Vo
---------------

## The Goal:

Working with images at scale can be challenging, especially when you need to extract useful information automatically. Recently, I explored a simple way to do this using AWS Lambda and AWS Rekognition, and I'd love to share what I learned.

In this post, I will walk through a straightforward, cost-effective, and scalable approach: using Lambda to read an image from an S3 bucket, analyze it with AWS Rekognition to detect objects, faces, or text, and then store the metadata in another S3 bucket.

## The How:

### Resources:

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

### Permissions and Roles:

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

### S3 Triggers:

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

## Deployment:

### AWS CLI:

I created an AWS CLI account with full permissions since I will deploy the Terraform files remotely from my local computer.

### Terraform CLI:

```sh
terraform fmt # Format main.tf to ensure consistency in syntax
terraform validate # Validate the configuration
terraform plan # Review what I'm about to deploy
terraform apply --auto-approve # Deploy main.tf
```

## Does It Work?

### Terraform Resource Monitoring:

After running `terraform apply`, I use this command to check how many resources are being managed by Terraform:

```sh
terraform state list
```

The test is straightforward. I upload a picture of a random animal into the input S3 bucket. The presence of a new object in this bucket triggers the Lambda function, which also logs an entry in the CloudWatch LogGroup. The Lambda function performs two tasks:

1. Makes a copy of the original picture, resizes it, and stores it in the output bucket.
2. Uses AWS Rekognition to analyze the image content and outputs the metadata as a JSON file into the output bucket.

It works perfectly, and here is the result.

## Refinement:

I have only scratched the surface of AWS Rekognition's use cases. This small application can be scaled up to handle a large volume of images in bulk.

## Resources and Credits
*Klayer https://github.com/keithrozario/Klayers*
*Rekognition Python https://docs.aws.amazon.com/rekognition/latest/customlabels-dg/ex-lambda.html*

