# lambda-ec2-dns

[ ![Codeship Status for indigoid/lambda-ec2-dns](https://app.codeship.com/projects/c7483940-8a4a-0134-69f7-5a7c9acf56e8/status?branch=master)](https://app.codeship.com/projects/184451)

Designed to be integrated with Codeship.

## Event flow

CloudWatch Rule -> Lambda -> Route53

## CloudFormation

Because plugging all the moving parts together by hand is painful, I've
put as much as possible into a CloudFormation template.

It takes three parameters:

- `HostedZoneId` which should have the Route53 hosted zone ID of the
  zone you want your new DNS entries to appear in. Eg. `Z354G969XV9QZ9`
- `HostedZoneName` which should have the domain name under which your
  DNS entries should be created. eg. `apse2.mydomain.com.`; note that
  this *must* have a trailing dot.
- `CodeshipIAMUser` which should have the name of the IAM user you want
  to use for autodeploying this code with Codeship.

When the template is deployed (see commands in the Stack Creation
section) the following resources are created:

- Codeship IAM policy (and attaches it to the specified user)
- IAM role for the Lambda function
- IAM policy for the Lambda function allowing it to do the things
- Lambda function
- CloudWatch Events Rule to trigger the Lambda function
- Production and Development aliases for the function

### Stack Creation

    # note here that the HostedZoneName parameter must have a trailing .
    # ie. "your.domain." *not* "your.domain"
    aws --region=your-region cloudformation create-stack \
      --stack-name asoe-autodns \
      --template-body file://asoe-autodns.json \
      --capabilities CAPABILITY_IAM \
      --parameters \
        ParameterKey=HostedZoneId,ParameterValue=YOUR_HOSTED_ZONE_ID \
        ParameterKey=HostedZoneName,ParameterValue=your.domain. \
        ParameterKey=CodeshipIAMUser,ParameterValue=svc_codeship

### Stack Update

    # note here that the HostedZoneName parameter must have a trailing .
    # ie. "your.domain." *not* "your.domain"
    aws --region your-region cloudformation update-stack \
      --stack-name asoe-autodns \
      --template-body file://asoe-autodns.json \
      --capabilities CAPABILITY_IAM \
      --parameters \
        ParameterKey=HostedZoneId,ParameterValue=YOUR_HOSTED_ZONE_ID \
        ParameterKey=HostedZoneName,ParameterValue=your.domain. \
        ParameterKey=CodeshipIAMUser,ParameterValue=svc_codeship

## Codeship Integration

More information on this can be found here:

https://blog.codeship.com/integrating-aws-lambda-with-codeship/

Note that to get this deploying with Codeship, you'll need to define the
below environment variables:

- `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` with the API
  key/secret for your Codeship IAM user
- `AWS_DEFAULT_REGION` with the region you created the CloudFormation
  stack in
- `LAMBDA_FUNCTION` with the contents of the `Lambda` output from the
  CloudFormation stack

## Todo

- get configuration from S3 instead
- don't lookup the hosted zone info every invocation because that's yuk
- optionally cache the DescribeInstance data in an Elasticache store for
  later use (such as by a related Lambda function that runs at instance
  termination...)

## Notes

Codeship/structure based on the work of my awesome colleague, Nathan
Humphreys.  Thanks!
