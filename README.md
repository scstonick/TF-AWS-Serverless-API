
# TF-AWS-Serverless-API

> **IMPORTANT**: When calling on API endpoints made with this module; if you are making requests from dev servers (like localhost or 127.0.0.1), then after creating the API via terraform, go into the AWS console and enable CORS on the API Gateway resources that you want to access. If you are using authentication with the API then you need to expand the advanced options when enabling CORS and set the value of Access-Control-Allow-Credentials to ``'true'`` (the ' before and after true are important)

## Description

This Terraform module can be used to quickly build, stage, and deploy an AWS API Gateway with configurable resources, methods, authorizers, and lambda functions with the option of layers.

## Examples

There are usage examples below with some explanations as well as a full example of how to use this with lambda layers included in the "example" folder of the repository.

## Basic usage
```
module  "api_gateway" { 
	source  =  "github.com/scstonick/T4-AWS-Serverless-API?ref=v1.0"
	api_name = "my-media-api"
	resource_names = ["image", "video"]
	stage_name = "init"
	functions =  [
		{
			func_zip_path = "./lambdafunctions/getImage/getImage.zip"
			func_name = "uploadImage"
			func_runtime = "nodejs18.x"
			func_role_arn = aws_iam_role.iam_for_first_lambda.arn
			method_type = "POST"
			region = "us-east-1"
			account_id = "YOUR_AWS_ACCOUNT_ID"
			resource_name = "image"
			layers = [ ]
		},
		{
			func_zip_path = "./lambdafunctions/getImage/getImage.zip"
			func_name = "getImage"
			func_runtime = "nodejs18.x"
			func_role_arn = aws_iam_role.iam_for_second_lambda.arn
			method_type = "GET"
			region = "us-east-1"
			account_id = "YOUR_AWS_ACCOUNT_ID"
			resource_name = "image"
			layers = [ ]
		},
		{
			func_zip_path = "./lambdafunctions/getVideo/getVideo.zip"
			func_name = "getVideo"
			func_runtime = "nodejs18.x"
			func_role_arn = aws_iam_role.iam_for_third_lambda.arn
			method_type = "GET"
			region = "us-east-1"
			account_id = "YOUR_AWS_ACCOUNT_ID"
			resource_name = "video"
			layers = [ ]
		}
	]
}
```

## Lambda layer usage
Each function in the "function array" has an variable that is an array called ``"layers"`` that can be configured to add layers to that specific lambda function. The base module has a variable called ``"unanimous_layers"`` that is an array that can be configured to add layers to every lambda function in the API. Both arrays use layer ARNs.
```
module  "api_gateway" { 
	source  =  "github.com/scstonick/T4-AWS-Serverless-API?ref=v1.0"
	api_name = "my-media-api"
	resource_names = ["image", "video"]
	stage_name = "init"
	unanimous_layers =  [ aws_lambda_layer_version.AWSLayer.arn ]
	functions =  [
		{
			func_zip_path = "./lambdafunctions/getImage/getImage.zip"
			func_name = "getImage"
			func_runtime = "nodejs18.x"
			func_role_arn = aws_iam_role.iam_for_lambda.arn
			method_type = "GET"
			region = "us-east-1"
			account_id = "YOUR_AWS_ACCOUNT_ID"
			resource_name = "image"
			layers = [ aws_lambda_layer_version.OtherAWSLayer.arn ]
		}
	]
}
```

## Auth usage
This module can be used in conjunction with an AWS Congito User Pool. If you set ``"user_pool_auth"`` to "1" and ``"cognito_user_pool_arn"`` to the arn of your user pool then the module will create an API Gateway Authorizer and implement that with all of your methods.
```
module  "api_gateway" { 
	source  =  "github.com/scstonick/T4-AWS-Serverless-API?ref=v1.0"
	api_name = "my-media-api"
	resource_names = ["image", "video"]
	stage_name = "init"
	unanimous_layers =  [ ]
	user_pool_auth =  1
	cognito_user_pool_arn = aws_cognito_user_pool.user_pool.arn
	functions =  [
		{
			func_zip_path = "./lambdafunctions/getImage/getImage.zip"
			func_name = "getImage"
			func_runtime = "nodejs18.x"
			func_role_arn = aws_iam_role.iam_for_lambda.arn
			method_type = "GET"
			region = "us-east-1"
			account_id = "YOUR_AWS_ACCOUNT_ID"
			resource_name = "image"
			layers = [ ]
		}
	]
}
```

## Technologies
This project is comprised of Terraform and Javascript files and it works with AWS API Gateway, AWS Lambda, and AWS IAM