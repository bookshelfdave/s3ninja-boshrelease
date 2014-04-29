=======
# BOSH Release for s3ninja

## Usage

To use this bosh release, first upload it to your bosh:

```
bosh target BOSH_HOST
git clone https://github.com/metadave/s3ninja-boshrelease.git
cd s3ninja-boshrelease
bosh upload release releases/s3ninja-1.yml
```

Create a `my-s3ninja.yml` input file to contain your AWS credentials:

```
properties:
  s3ninja:
    aws_access_key: admin
    aws_secret_key: SECRET_KEY
```

For [bosh-lite](https://github.com/cloudfoundry/bosh-lite), you can quickly create a deployment manifest, using the keys above, and deploy a VM:

```
templates/make_manifest warden my-s3ninja.yml
bosh -n deploy
```

For AWS EC2, create a single VM backed by an EBS:

```
templates/make_manifest aws-ec2 my-s3ninja.yml
bosh -n deploy
```


### Properties

S3Ninja defaults to port 9444, which you can override in the properties file.

Property name | Description | Default
--------------|-------------|---------
s3ninja.aws_access_key|AWS Access Key| no default, **REQUIRED**
s3ninja.aws_secret_key|AWS Secret Key| no default, **REQUIRED**
s3ninja.autocreate_buckets|if `true`, buckets wil be auto created on the first request via the S3 API|`true`
s3ninja.uiport|The port that the S3ninja gui runs on|9444
s3ninja.s3port|S3 endpoint port. See below for more info| 9445
nginx.proxy_timeout_seconds|Proxy send/read timeout|10

####S3Ninja gui

The S3Ninja gui litens on `s3ninja.uiport`. 

####S3Ninja/Nginx proxy

Because S3Ninja uses /s3 as the S3 endpoint, many S3 clients can't connect to it directly. This release contains an Nginx proxy that listens on a separate port (`s3ninja.s3port`) to service commands at /. 

### Override security groups

For AWS & Openstack, the default deployment assumes there is a `default` security group. If you wish to use a different security group(s) then you can pass in additional configuration when running `make_manifest` above.

Create a file `my-networking.yml`:

``` yaml
---
networks:
  - name: s3ninja1
    type: dynamic
    cloud_properties:
      security_groups:
        - s3ninja
```

Where `- s3ninja` means you wish to use an existing security group called `s3ninja`.

You now suffix this file path to the `make_manifest` command:

```
templates/make_manifest openstack-nova my-networking.yml
bosh -n deploy
```


### bosh blobstore_client_console

To use the blobstore client provided with bosh, create a file called `blobstore-s3ninja.yml` using the following as a template:

```yaml
---
bucket_name: foobar
access_key_id: admin
secret_access_key: SECRET_KEY
use_ssl: false
port: 9445
host: 10.244.2.2
s3_force_path_style: true
```

Next, pass the file you just created to the `blobstore_client_console` command:

```
blobstore_client_console -p s3 -c blobstore-s3ninja.yml
oid = bsc.create("test data content")
bsc.get(oid)
bsc.delete(oid)
```

