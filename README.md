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


## Install and run the CloudWatch agent manually

The instructions below are used to install the CloudWatch agents on the EC2
instances to collect metrics such as CPU, memory, disk and network usage.

1. Ssh into the EC2 instance.
2. [Download the CloudWatch agent for Ubuntu (x86-64)].

    ```
    wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
    ```

3. Install the agent.

    ```
    sudo dpkg -i -E ./amazon-cloudwatch-agent.deb
    ```

4. Download the CloudWatch agent configuration.

    ```
    wget https://github.com/nlpsandbox/nlpsandbox-infra/blob/main/cloudwatch-config.json
    ```

5. Start the CloudWatch agent.

    ```
    sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:cloudwatch-config.json -s
    ```

6. Check the start of the CloudWatch agent.

    ```
    sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
    ```

To stop the agent:

```
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a stop
```

[Download the CloudWatch agent for Ubuntu (x86-64)]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/download-cloudwatch-agent-commandline.html