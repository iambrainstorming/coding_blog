# Upload files safely (tested)


```javascript
import dotenv from 'dotenv';
import { S3 } from "@aws-sdk/client-s3";
import { Upload } from "@aws-sdk/lib-storage";

dotenv.config();


// Function to upload a file using temporary credentials
async function uploadFileWithTemporaryCredentials(file) {
    const s3 = new S3({
      endpoint: 'https://endpoint.4everland.co',
      signatureVersion: 'v4',
      credentials: {
        accessKeyId: process.env.AccessKeyId,
        secretAccessKey: process.env.SecretAccessKey,
        sessionToken: process.env.SessionToken,
      },
      region: 'us-west-2',
    });
  
    const params = {
      Bucket: 'shivarthu-upload', // Bucket name
      Key: 'website_tests/' + file.name, // folder name + file name
      Body: file.inside_text, // Inside text
      ContentType: file.type,
    };
  
    try {
      const task = new Upload({
        client: s3,
        queueSize: 3, // 3 MiB
        params,
      });
  
      task.on('httpUploadProgress', (e) => {
        const progress = ((e.loaded / e.total) * 100) | 0;
        console.log(progress, e);
      });
  
      await task.done();
    } catch (error) {
      console.error('Error uploading file:', error.message);
    }
  }

// Example usage
async function main() {
    try {  
      // Upload file using temporary credentials
      const file = {
        name: 'example.txt',
        type: 'text/plain',
        inside_text: "Website tests",
        // ... other file properties
      };
  
      await uploadFileWithTemporaryCredentials(file);
  
      console.log('File uploaded successfully!');
    } catch (error) {
      console.error('Error:', error.message);
    }
  }
  
  main();
```

## .env file in root directory

Change it with original key and secret generated from sts

```
AccessKeyId=ldjfei
SecretAccessKey=eriueir
SessionToken=i9erijdfj
Expiration=2024-03-23T21:27:36.000Z
```