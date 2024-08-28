
write terraform to create a python lambda that writes "hello world" to cloudwatch logs, a clodwatch schedule to run it every 24 hours, a cloudwatch alert that sends an email noticifaction when it detect "hello world" in the cloudwatch log, and an cloudwatch
	alert that sends an email notification if the python lambda doesn't run. Tag everything with "matt_tf", adn acceept email address & cloudwatch schedule are input variables


GitHub Copilot
Terraform Implementation


```shell
variable "email_address" {
	description = "Email address for SNS notifications"
	type        = string
}


variable "cloudwatch_schedule" {
	description = "CloudWatch schedule expression"
	type        = string
	default     = "rate(24 hours)"
}


provider "aws" {
	region = "us-west-2"
}


resource "aws_iam_role" "lambda_role" {
	name = "lambda_execution_role"


	assume_role_policy = jsonencode({
	  Version = "2012-10-17"
	  Statement = [
	    {
	      Action = "sts:AssumeRole"
	      Effect = "Allow"
	      Principal = {
	        Service = "lambda.amazonaws.com"
	      }
	    }
	  ]
	})


	tags = {
	  Name = "matt_tf"
	}
}


resource "aws_iam_role_policy" "lambda_policy" {
	name = "lambda_policy"
	role = aws_iam_role.lambda_role.id


	policy = jsonencode({
	  Version = "2012-10-17"
	  Statement = [
	    {
	      Action = [
	        "logs:CreateLogGroup",
	        "logs:CreateLogStream",
	        "logs:PutLogEvents"
	      ]
	      Effect = "Allow"
	      Resource = "*"
	    },
	    {
	      Action = "events:PutEvents"
	      Effect = "Allow"
	      Resource = "*"
	    }
	  ]
	})


	tags = {
	  Name = "matt_tf"
	}
}


resource "aws_lambda_function" "hello_world" {
	filename         = "lambda_function_payload.zip"
	function_name    = "HelloWorldLogger"
	role             = aws_iam_role.lambda_role.arn
	handler          = "index.lambda_handler"
	runtime          = "python3.8"
	source_code_hash = filebase64sha256("lambda_function_payload.zip")


	environment {
	  variables = {
	    LOG_LEVEL = "INFO"
	  }
	}


	tags = {
	  Name = "matt_tf"
	}
}


resource "aws_cloudwatch_event_rule" "every_24_hours" {
	name                = "DailyHelloWorldLogger"
	schedule_expression = var.cloudwatch_schedule


	tags = {
	  Name = "matt_tf"
	}
}


resource "aws_cloudwatch_event_target" "lambda_target" {
	rule      = aws_cloudwatch_event_rule.every_24_hours.name
	target_id = "HelloWorldLogger"
	arn       = aws_lambda_function.hello_world.arn


	tags = {
	  Name = "matt_tf"
	}
}


resource "aws_lambda_permission" "allow_cloudwatch" {
	statement_id  = "AllowExecutionFromCloudWatch"
	action        = "lambda:InvokeFunction"
	function_name = aws_lambda_function.hello_world.function_name
	principal     = "events.amazonaws.com"
	source_arn    = aws_cloudwatch_event_rule.every_24_hours.arn


	tags = {
	  Name = "matt_tf"
	}
}


resource "aws_cloudwatch_log_metric_filter" "hello_world_filter" {
	name           = "HelloWorldFilter"
	log_group_name = "/aws/lambda/HelloWorldLogger"
	pattern        = "\"hello world\""


	metric_transformation {
	  name      = "HelloWorldCount"
	  namespace = "HelloWorldNamespace"
	  value     = "1"
	}


	tags = {
	  Name = "matt_tf"
	}
}


resource "aws_sns_topic" "hello_world_topic" {
	name = "HelloWorldDetected"


	tags = {
	  Name = "matt_tf"
	}
}


resource "aws_sns_topic" "lambda_not_run_topic" {
	name = "LambdaNotRun"


	tags = {
	  Name = "matt_tf"
	}
}


resource "aws_sns_topic_subscription" "hello_world_subscription" {
	topic_arn = aws_sns_topic.hello_world_topic.arn
	protocol  = "email"
	endpoint  = var.email_address


	tags = {
	  Name = "matt_tf"
	}
}


resource "aws_sns_topic_subscription" "lambda_not_run_subscription" {
	topic_arn = aws_sns_topic.lambda_not_run_topic.arn
	protocol  = "email"
	endpoint  = var.email_address


	tags = {
	  Name = "matt_tf"
	}
}


resource "aws_cloudwatch_metric_alarm" "hello_world_alarm" {
	alarm_name          = "HelloWorldDetectedAlarm"
	comparison_operator = "GreaterThanOrEqualToThreshold"
	evaluation_periods  = "1"
	metric_name         = "HelloWorldCount"
	namespace           = "HelloWorldNamespace"
	period              = "300"
	statistic           = "Sum"
	threshold           = "1"
	alarm_actions       = [aws_sns_topic.hello_world_topic.arn]


	tags = {
	  Name = "matt_tf"
	}
}


resource "aws_cloudwatch_metric_alarm" "lambda_not_run_alarm" {
	alarm_name          = "LambdaNotRunAlarm"
	comparison_operator = "LessThanThreshold"
	evaluation_periods  = "1"
	metric_name         = "Invocations"
	namespace           = "AWS/Lambda"
	period              = "86400"
	statistic           = "Sum"
	threshold           = "1"
	alarm_actions       = [aws_sns_topic.lambda_not_run_topic.arn]


	dimensions = {
	  FunctionName = aws_lambda_function.hello_world.function_name
	}


	tags = {
	  Name = "matt_tf"
	}
}
```


Notes
Replace YOUR_EMAIL@example.com with your actual email address.
Ensure that the Lambda function code is packaged into a lambda_function_payload.zip file and placed in the same directory as the Terraform configuration. The index.py file inside the zip should contain the Lambda function code:
```python
import logging


def lambda_handler(event, context):
	  logger = logging.getLogger()
	  logger.setLevel(logging.INFO)
	  logger.info("hello world")
```#   t e r r a f o r m _ l a m b d a _ l o g _ n o t i f y  
 