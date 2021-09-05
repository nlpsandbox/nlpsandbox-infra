# NLP Sandbox CloudFormation template

AWS CloudFormation template for deploying the NLP Sandbox benchmarking infrastructure

## Pre-commit

```console
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

    ```console
    wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
    ```

3. Install the agent.

    ```console
    sudo dpkg -i -E ./amazon-cloudwatch-agent.deb
    ```

4. Download the CloudWatch agent configuration.

    ```console
    wget https://github.com/nlpsandbox/nlpsandbox-infra/blob/main/cloudwatch-config.json
    ```

5. Start the CloudWatch agent.

    ```console
    sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:cloudwatch-config.json -s
    ```

6. Check the start of the CloudWatch agent.

    ```console
    sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
    ```

To stop the agent:

```console
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a stop
```

## Pushing logs to CloudWatch

### Requirements

- [Configure your IAM role or user for CloudWatch Logs]

### Creating the log group and log stream

Create a log group named `/var/log/syslog` that will gather logs from the file
with the same name.

```console
aws logs create-log-group --log-group-name /var/log/syslog
```

Create a log stream per file or type of log files and add it to an existing log
group. Here we name the stream following the name of the instance
(`controller`).

```console
aws logs create-log-stream \
  --log-group-name /var/log/syslog \
  --log-stream-name controller
```

Send messages to a log stream for testing. These messages should then be visible
in `AWS Console` > `CloudWatch` > `Log groups`.

```console
aws logs put-log-events \
  --log-group-name /var/log/syslog \
  --log-stream-name controller \
  --log-events \
    timestamp=1630159633000,message="This message contains an Error" \
    timestamp=1630159633000,message="checking progress or starting new job"
```

Another way of testing that syslog message reach AWS is by printing a message to
syslog.

```console
echo -e "This is a test message captured by syslog" | tee >(exec logger)
```

### Creating a metric filter

Create a log filter that listen to the log group `/var/log/syslog`.

```console
aws logs put-metric-filter \
  --log-group-name /var/log/syslog \
  --filter-name ErrorCount \
  --filter-pattern 'Error' \
  --metric-transformations \
      metricName=Count,metricNamespace=MyNamespace,metricValue=1,defaultValue=0
```

The pattern value is case sensitive. Also, the error priority defined by the
syslog file is "err".

```console
aws logs put-metric-filter \
  --log-group-name /var/log/syslog \
  --filter-name errCount \
  --filter-pattern 'err' \
  --metric-transformations \
      metricName=Count,metricNamespace=MyNamespace,metricValue=1,defaultValue=0
```

### Pushing log files to CloudWatch

In order to read the content of `/var/syslog` to CW, add the user `cwagent` that
runs the agent to the group `adm`.

```console
sudo usermod -a -G adm cwagent
```

List in the section "logs" of the configuration file of the CW agent the files
whose content need to be pushed to CW.

```json
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log",
            "log_group_name": "amazon-cloudwatch-agent.log",
            "log_stream_name":"controller"
          },
          {
            "file_path": "/var/log/syslog",
            "log_group_name": "/var/log/syslog",
            "log_stream_name":"controller"
          }
        ]
      }
    }
  }
```

### Pushing container logs

Create the log group:

```console
aws logs create-log-group --log-group-name docker-logs
```

Create the log stream:

```console
aws logs create-log-stream \
  --log-group-name docker-logs \
  --log-stream-name controller
```

Start a container with the log driver `awslogs`:

```console
docker run --rm \
  --log-driver=awslogs \
  --log-opt awslogs-group=docker-logs \
  --log-opt awslogs-stream=controller \
  alpine echo 'Test message'
```

<!-- Links -->

[Download the CloudWatch agent for Ubuntu (x86-64)]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/download-cloudwatch-agent-commandline.html
[Configure your IAM role or user for CloudWatch Logs]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/QuickStartEC2Instance.html