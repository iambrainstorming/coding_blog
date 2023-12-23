# Create a custom policy

To restrict specific users from deleting an S3 bucket, you can create a custom policy using the 4EVER Security Token Service (STS) API to generate temporary credentials with limited permissions. Below are the general steps to achieve this:

### Step 1: Create a Custom Policy

Define a policy that allows only specific actions, such as read-only access, for your S3 bucket. In this example, I'm assuming you want to restrict users from deleting the bucket.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:GetObject",
                "s3:GetObjectVersion"
            ],
            "Resource": [
                "arn:aws:s3:::your-bucket-name",
                "arn:aws:s3:::your-bucket-name/*"
            ]
        },
        {
            "Effect": "Deny",
            "Action": "s3:DeleteBucket",
            "Resource": "arn:aws:s3:::your-bucket-name"
        }
    ]
}
```


## Put data

```json
{
  "Id": "Policy1703314728428",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1703314504947",
      "Action": [
        "s3:PutObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::your-bucket-name",
      "Principal": {
        "AWS": [
          "allow to write"
        ]
      }
    },
    {
      "Sid": "Stmt1703314726015",
      "Action": [
        "s3:DeleteBucket"
      ],
      "Effect": "Deny",
      "Resource": "arn:aws:s3:::your-bucket-name",
      "Principal": {
        "AWS": [
          "deny delete bucket"
        ]
      }
    }
  ]
}
```

Replace `"your-bucket-name"` with the actual name of your S3 bucket.

### Step 2: Use 4EVER STS API to Generate Temporary Credentials

Use the 4EVER STS API to generate temporary credentials based on the custom policy. Here's an example using the JavaScript SDK:

```javascript
const { STSClient, AssumeRoleCommand } = require('@aws-sdk/client-sts');

const stsClient = new STSClient({
    endpoint: 'https://endpoint.4everland.co',
    region: 'us-west-1',
    credentials: {
        accessKeyId: 'Your api key.',
        secretAccessKey: 'Your api secret.',
    }
});

const params = {
    RoleSessionName: "read-only-user",
    DurationSeconds: 3600,  // Adjust as needed
    Policy: JSON.stringify(/* Your custom policy */),
};

stsClient.send(new AssumeRoleCommand(params)).then(function (data) {
    // Use data.Credentials for your S3 operations
    console.log("Temporary Credentials:", data.Credentials);
});
```

### Step 3: Use Temporary Credentials with AWS SDK for S3

Replace the AWS SDK for S3 credentials with the temporary credentials obtained from the 4EVER STS API. Ensure your S3 operations use these temporary credentials.

Remember that this is a basic example, and you might need to adjust the policy according to your specific requirements. Always follow the principle of least privilege when defining policies to ensure that users have only the permissions they need.

[IAM JSON policy elements reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html)


[AWS Policy Generator](https://awspolicygen.s3.amazonaws.com/policygen.html)