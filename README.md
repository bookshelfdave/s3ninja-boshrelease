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

Createa `my-s3ninja.yml` input file to contain your AWS credentials:

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
port|The port that s3ninja listens on|9444
aws_access_key|AWS Access Key| no default, **REQUIRED**
aws_secret_key|AWS Secret Key| no default, **REQUIRED**
autocreate_buckets|if `true`, buckets wil be auto created on the first request via the S3 API|`true`

See `./examples/default.yml` for more info.

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
