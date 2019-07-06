---
date: 2019-07-05T15:00:00-06:00
title: Odd Attribute Error from AWS boto3 
commentIssueId: 17
tags: 
  - python
  - aws
math: false
---

I've been playing around with a side project that makes use of some serverless [AWS Lambda](https://aws.amazon.com/lambda/) functions. I used a simple Flask app locally for testing, which took the place of the [API Gateway](https://aws.amazon.com/api-gateway/)'s role of getting server requests to the right function, and locally everything was testing fine. Note that within the Python AWS environment, the AWS credentials consisting of an Access Key ID and Secret Access Key are found as environment variables. In order to replicate this locally, I was creating environment variables with the Flask server, and then my functions would access these keys the same way:

```python
import boto3
# Other imports....

# === Determine if we're running in a Lambda function. =============================================
try:
    _ = os.environ['AWS_REGION']
    in_lambda = True
except KeyError:
    # Not in Lambda environment
	in_lambda = False

secrets = {
    "AWS_ACCESS_KEY_ID": os.environ["AWS_ACCESS_KEY_ID"],
    "AWS_SECRET_ACCESS_KEY": os.environ["AWS_SECRET_ACCESS_KEY"]
}

if in_lambda:
    secrets["AWS_SESSION_TOKEN"] = os.environ["AWS_SESSION_TOKEN"],
    aws_session = boto3.session.Session(aws_access_key_id=secrets["AWS_ACCESS_KEY_ID"],
                                        aws_secret_access_key=secrets["AWS_SECRET_ACCESS_KEY"],
                                        aws_session_token=secrets["AWS_SESSION_TOKEN"],
                                        region_name='us-east-2')
else:
    aws_session = boto3.session.Session(aws_access_key_id=secrets["AWS_ACCESS_KEY_ID"],
                                        aws_secret_access_key=secrets["AWS_SECRET_ACCESS_KEY"],
										region_name='us-east-2')


# Do stuff
s3 = aws_session.client('s3')

```

Locally, this was working great, but when I went to test it on AWS, I was getting the following exception:

```python
AttributeError: 'tuple' object has no attribute 'strip'
```

Digging through the exception, I found that the root of the exception came from the `botocore` package within the `botocore/auth.py` file at line 381[^1], when the package is attempting to form a request and uses `self.credential.token` to form the `X-Amz-Security-Token`. For some reason that I'm still not certain of, the above snippet would pass in the credentials as a tuple instead of a string, which would then eventually have the string method `strip` called on it to create the header value, and subsequently raise the exception.

Changing to the following has since mitigated the problem. Again, I'm not quite sure why yet, but I spent too much time as it was on this and I've currently left it here. Feel free to let me know what I've missed, since I'm certainly very new with using AWS Lambda and definitely could be missing something obvious or trivial.

```python
import boto3
# Other imports....

# === Determine if we're running in a Lambda function. =============================================
try:
    _ = os.environ['AWS_REGION']
    in_lambda = True
except KeyError:
    # Not in Lambda environment
    in_lambda = False


# === Load required environment variables and define AWS session ===================================
if in_lambda:
    aws_session = boto3
    secrets = {
        # Other secrets...
    }
else:
    secrets = {
        "AWS_ACCESS_KEY_ID": os.environ["AWS_ACCESS_KEY_ID"],
        "AWS_SECRET_ACCESS_KEY": os.environ["AWS_SECRET_ACCESS_KEY"]
    }

    aws_session = boto3.session.Session(aws_access_key_id=secrets["AWS_ACCESS_KEY_ID"],
                                        aws_secret_access_key=secrets["AWS_SECRET_ACCESS_KEY"],
										region_name='us-east-2')

# Do stuff
s3 = aws_session.client('s3')

```

[^1]: [GitHub: boto/botocore/auth.py](https://github.com/boto/botocore/blob/526f1e0322bf87e74f69c894245ce8506b86e597/botocore/auth.py#L388)