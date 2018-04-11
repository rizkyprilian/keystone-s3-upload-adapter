# Digital Ocean Space storage adapter for KeystoneJS

[![NPM](https://nodei.co/npm/keystone-s3-upload-adapter.png)](https://nodei.co/npm/keystone-s3-upload-adapter/)

This adapter is designed to replace the existing `S3File` field in KeystoneJS using the new storage API.

Compatible with Node.js 0.12+

## Usage

Install Package:
```
npm install --save keystone-dospace-upload-adapter
```

Configure the storage adapter:

```js
var s3Storage = new keystone.Storage({
    adapter: require('keystone-s3-upload-adapter'),
    s3: {
        key: 's3-key', // required; defaults to process.env.S3_KEY
        secret: 'secret', // required; defaults to process.env.S3_SECRET
        bucket: 'bucket', // required; defaults to process.env.S3_BUCKET
        region: 'region', // optional; defaults to process.env.S3_REGION, or if that's not specified, us-east-1
        endpoint: 's3.us-east-1.amazonaws.com' // to use with digitalocean space, alter this endpoint to match with digitalocean space configuration
        path: 'images',
        headers: {
            'x-amz-acl': 'public-read', // add default headers; see below for details
        },
    },
    schema: {
        bucket: true, // optional; store the bucket the file was uploaded to in your db
        etag: true, // optional; store the etag for the resource
        path: true, // optional; store the path of the file in your db
        url: true, // optional; generate & store a public URL
    },
});
```

Use it as a type in Keystone Field (Example Below):

```js
imageUpload: {
        type: Types.File,
        storage: s3Storage,
        filename: function (item, file) {
            return encodeURI(item._id + '-' + item.name);
        },
    },
```

### Options:

The adapter requires an additional `s3` field added to the storage options. It accepts the following values:

- **key**: *(required)* AWS access key. Configure your AWS credentials in the [IAM console](https://console.aws.amazon.com/iam/home?region=ap-southeast-2#home).

- **secret**: *(required)* AWS access secret.

- **bucket**: *(required)* S3 bucket to upload files to. Bucket must be created before it can be used. Configure your bucket through the AWS console [here](https://console.aws.amazon.com/s3/home?region=ap-southeast-2).

- **region**: AWS region to connect to. AWS buckets are global, but local regions will let you upload and download files faster. Defaults to `'us-standard'`. Eg, `'us-west-2'`.

- **path**: Storage path inside the bucket. By default uploaded files will be stored in the root of the bucket. You can override this by specifying a base path here. Path can be either absolute, for example '/images/profilepics', or relative, for example 'images/profilepics'.

- **headers**: Default headers to add when uploading files to S3. You can use these headers to configure lots of additional properties and store (small) extra data about the files in S3 itself. See [AWS documentation](http://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectPUT.html) for options. Examples: `{"x-amz-acl": "public-read"}` to override the bucket ACL and make all uploaded files globally readable.


### Schema

The S3 adapter supports all the standard Keystone file schema fields. It also supports storing the following values per-file:

- **bucket**, **path**: The bucket, and path within the bucket, for the file can be is stored in the database. If these are present when reading or deleting files, they will be used instead of looking at the adapter configuration. The effect of this is that you can have some (eg, old) files in your collection stored in different bucket / different path inside your bucket.

The main use of this is to allow slow data migrations. If you *don't* store these values you can arguably migrate your data more easily - just move it all, then reconfigure and restart your server.

- **etag**: The etag of the stored item. This is equal to the MD5 sum of the file content.
