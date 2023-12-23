

[Bucket Api](https://docs.4everland.org/storage/bucket/bucket-api-s3-compatible/coding-examples/example-for-javascript)

```bash
npm install @aws-sdk/client-s3
```

# Example

## init s3 client

```javascript
import { S3 } from "@aws-sdk/client-s3";

const { endpoint, accessKey, secretKey, sessionToken } = s3Params;
const s3 = new S3({
  endpoint,
  signatureVersion: "v4",
  credentials: {
    accessKeyId: accessKey,
    secretAccessKey: secretKey,
    sessionToken,
  },
  region: "us-west-2",
});
```

##  create bucket and put object

```javascript
import { Upload } from "@aws-sdk/lib-storage";

const createBucketOutput = await s3.createBucket({
  Bucket: "bucketname",
});
const putObjectOutput = await s3.putObject({
  Bucket: "bucketname",
  Key: "key",
  Body: "data content",
});

// multipart upload
const params = {
  Bucket,
  Key: file.name,
  Body: file,
  ContentType: file.type,
};
try {
  const task = new Upload({
    client: s3,
    queueSize: 3, // 3 MiB
    params,
  });
  task.on("httpUploadProgress", (e) => {
    const progress = ((e.loaded / e.total) * 100) | 0;
    console.log(progress, e);
  });
  await task.done();
} catch (error) {
  if (error) {
    console.log("task", error.message);
  }
}
```

## List buckets and list objects

```javascript
s3.listBuckets((err, data)=> {
    if(!err) {
        console.log(data)
    }
});

s3.listObjectsV2({ Bucket: "bucketname", MaxKeys: 10 }, (err, data)=>{
    if(!err) {
        console.log(data.KeyCount)
        data.Contents.forEach((o)=>{
            console.log(o.Key)
        })
        
    }
});
```

## Get ipfs hash and arweave hash

```javascript
s3.headObject({
    Bucket: "bucketname",
    Key: "objectkey",
},(err, data)=>{
    if(!err) {
        console.log(data)
        console.log("ipfs cid:    ", data.Metadata['ipfs-hash'])
        // if synced to arweave
        console.log("arweave hash:", data.Metadata['arweave-hash']) 
    }

})
```



