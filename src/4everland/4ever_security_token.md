# 4EVER Security Token Service API

Welcome to the 4EVERLAND Security Token Service (`STS`) API
## Introduction
The 4EVERLAND Security Token Service is a service designed to generate temporary credentials, allowing you to request temporary credentials for partial or even full privileges for other users or a specified user.
4EVER Bucket STS fully compatible with AWS S3 STS. More information about STS can be found at .


## How to use
This is obtained by taking the API Secret and the API Key from the Master key and sending them to the endpoint. By default, all 4EVERLAND STS requests are sent to https://endpoint.4everland.co.


Javascript example:
```javascript

const {STSClient, AssumeRoleCommand} = require('@aws-sdk/client-sts')
​
const stsClient = new STSClient({
    endpoint: 'https://endpoint.4everland.co',
    region: 'us-west-1',
    credentials: {
        accessKeyId: 'Your api key.',
        secretAccessKey:  'Your api secret.',
    }
});
​
const params = {
    RoleSessionName: "only-put-object",
    DurationSeconds: 3600,
    Policy: `{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:AbortMultipartUpload"
            ],
            "Resource": [
                "arn:aws:s3:::your-bucket-name/your-directory/*"
            ]
        }
    ]
}`
};
​
​
stsClient.send(new AssumeRoleCommand(params)).then(function (data) {
    console.log(data.Credentials)
});

```
## How to get the Master Key

Click on Bucket> Access key> Master-Key

## Session token according to chat gpt

```javascript
stsClient.send(new AssumeRoleCommand(params)).then(function (data) {
    // Extract the session token from the temporary credentials
    const sessionToken = data.Credentials.SessionToken;
    console.log("Session Token:", sessionToken);
});
```

## STS in one function

To create an export function that takes in `accessKeyId` and `secretAccessKey` as arguments and returns the credentials, you can modify the code like this:

```javascript
const { STSClient, AssumeRoleCommand } = require("@aws-sdk/client-sts");

async function getSTSCredentials(accessKeyId, secretAccessKey) {
    const stsClient = new STSClient({
        endpoint: 'https://endpoint.4everland.co',
        region: 'us-west-1',
        credentials: {
            accessKeyId: accessKeyId,
            secretAccessKey: secretAccessKey,
        }
    });

    const params = {
        RoleSessionName: "only-put-object",
        DurationSeconds: 3600,
        Policy: `{
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Action": [
                        "s3:PutObject",
                        "s3:AbortMultipartUpload"
                    ],
                    "Resource": [
                        "arn:aws:s3:::your-bucket-name/your-directory/*"
                    ]
                }
            ]
        }`
    };

    try {
        const data = await stsClient.send(new AssumeRoleCommand(params));
        return data.Credentials;
    } catch (error) {
        console.error("Error fetching STS credentials:", error);
        throw error;
    }
}

// Example usage:
const accessKeyId = 'Your api key.';
const secretAccessKey = 'Your api secret.';

getSTSCredentials(accessKeyId, secretAccessKey)
    .then(credentials => {
        console.log(credentials);
    })
    .catch(error => {
        console.error("Error:", error);
    });
```

This code defines an `async` function `getSTSCredentials` that takes `accessKeyId` and `secretAccessKey` as arguments, creates an STS client, sends an `AssumeRoleCommand`, and returns the resulting credentials. The example usage section demonstrates how to call this function with your access key and secret key.

## Compiled code (tested)

app.js

```javascript
import dotenv from 'dotenv';
import {STSClient, AssumeRoleCommand } from "@aws-sdk/client-sts";


dotenv.config();


// shivarthu-upload/website_tests/*
// Bucket name `shivarthu-upload`
// Folder name `website_test`

async function getSTSCredentials(accessKeyId, secretAccessKey) {
    const stsClient = new STSClient({
        endpoint: 'https://endpoint.4everland.co',
        region: 'us-west-1',
        credentials: {
            accessKeyId: accessKeyId,
            secretAccessKey: secretAccessKey,
        }
    });

    const params = {
        RoleSessionName: "only-put-object",
        DurationSeconds: 7889400, // 3 months
        Policy: `{
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Action": [
                        "s3:PutObject",
                        "s3:AbortMultipartUpload"
                    ],
                    "Resource": [
                        "arn:aws:s3:::shivarthu-upload/website_tests/*" 
                    ]
                }
            ]
        }`
    };

    try {
        const data = await stsClient.send(new AssumeRoleCommand(params));
        return data.Credentials;
    } catch (error) {
        console.error("Error fetching STS credentials:", error);
        throw error;
    }
}

// Example usage:
const accessKeyId = process.env.EVERLAND_KEY_MASTER;
const secretAccessKey = process.env.EVERLAND_SECRET_MASTER;

getSTSCredentials(accessKeyId, secretAccessKey)
    .then(credentials => {
        console.log(credentials);
    })
    .catch(error => {
        console.error("Error:", error);
    });

```

## .env file in root directory

Change it with original master key and secret

```
EVERLAND_KEY_MASTER=kdifeiikd
EVERLAND_SECRET_MASTER=ierudieikdie
```