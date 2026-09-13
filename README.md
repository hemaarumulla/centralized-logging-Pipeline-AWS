# centralized-logging-Pipeline-AWS
ec2(/var/log/*)->cloud watch->event bridge->lambda->s3->lifecyclerule
#  Install the CloudWatch Agent
```bah
sudo yum install amazon-cloudwatch-agent -y
```
# Create or Edit the Configuration File
```bash
sudo vi /opt/aws/amazon-cloudwatch-agent/bin/config.json
```
```bash
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/*",
            "log_group_name": "LOG-FROM-EC2",
            "log_stream_name": "{instance_id}",
            "retention_in_days": 1
          }
        ]
      }
    }
  }
}
```
⚠️ Replace LOG-FROM-EC2 with your custom log group name if needed.

# Start the CloudWatch Agent
```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
-a fetch-config \
-m ec2 \
-c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json \
-s
```

- Verify Agent is Running
 ```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
```


# 🔐 Add Bucket Policy

```bash
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "logs.<REGION>.amazonaws.com"
            },
            "Action": "s3:GetBucketAcl",
            "Resource": "arn:aws:s3:::<BUCKET_NAME>"
        },
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "logs.<REGION>.amazonaws.com"
            },
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::<BUCKET_NAME>/*",
            "Condition": {
                "StringEquals": {
                    "s3:x-amz-acl": "bucket-owner-full-control",
                    "aws:SourceAccount": "<ACCOUNT_ID>"
                }
            }
        }
    ]
}
```

## ⚠️ Replace the following values

* `<BUCKET_NAME>` → Your S3 bucket name where CloudWatch logs will be stored
* `<ACCOUNT_ID>` → Your AWS Account ID
* `<REGION>` → AWS region of CloudWatch Logs

### Example

```bash
REGION=ap-south-1
BUCKET_NAME=my-cloudwatch-logs
ACCOUNT_ID=123456789012
```

