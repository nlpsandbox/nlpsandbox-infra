# aws-cloudformation
AWS CloudFormation template for deploying the NLP Sandbox benchmarking infrastructure


## Pre-commit

```
# Install pre-commit
pre-commit install
# Run pre-commit
pre-commit run --all-files
```


## S3 static website / redirections

While most of the S3 configuration is included in sceptre/nlpsandbox/templates/s3.yaml, but the resource that needs to be configured manually is the cloudfront distribution.  Follow these [AWS instructions](https://aws.amazon.com/premiumsupport/knowledge-center/cloudfront-https-requests-s3/) to serve HTTPS requests for your S3 bucket.

- When updating redirections on the S3 bucket, you may have to clear the cloudfront cache.  Follow [this stackoverflow solution](https://stackoverflow.com/questions/22021651/amazon-s3-and-cloudfront-cache-how-to-clear-cache-or-synchronize-their-cache/63238713#63238713).
