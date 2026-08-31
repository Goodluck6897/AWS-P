Create a user s3-assume-user


Create a role with trust policy Role 

Name:Role-For-S3-assume-user
```yaml
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::019419900643:user/s3-assume-user"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

Add S3 Full access to the role  Role-For-S3-assume-user

Login to the user S3-assume-user and switch role provide Role-For-S3-assume-user
