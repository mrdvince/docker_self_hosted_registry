
# Docker self hosted registry

> Why host you own registry? Well it's free"ish" compared to hosting your images on docker hub, plus you can just use cloud credits if you have them. Now you can just pull your own images into your Kubernetes cluster etc.

## Getting started

1. Clone the repo and cd into the cloned folder.

```bash
  https://github.com/mrdvince/docker_self_hosted_registry.git
  cd docker_self_hosted_registry
```

2. Generate a password

```bash
 docker run \
  --entrypoint htpasswd \
  httpd:2 -Bbn registry regtoor > auth/htpasswd
```

> docker registry supports other auth methods you can check those out in the docker docs.

3. Create an AWS S3 bucket, user & policy

3a. Create an AWS bucket

3b. Create a policy, copy the below json replacing the `bucket_name` with your bucket name

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListAllMyBuckets"
            ],
            "Resource": "arn:aws:s3:::*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:GetBucketLocation",
                "s3:ListBucketMultipartUploads"
            ],
            "Resource": "arn:aws:s3:::<bucket_name>"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:DeleteObject",
                "s3:ListMultipartUploadParts",
                "s3:AbortMultipartUpload"
            ],
            "Resource": "arn:aws:s3:::<bucket_name>/*"
        }
    ]
}
```
3c. Assign this policy to the IAM user created.

4. Download user creds and replace the fields in the `config.yml` file

5. Start the services
```bash
    docker-compose up -d
```
## Usage/Testing

1. Login to the registry

```bash
docker login <registry domain name>
```

2. Pull an image from docker hub, tag it and push it to the self hosted registry

```bash
# pull an image from docker hub 
docker pull ubuntu

#  tag the image so that it points to the self hosted registry
docker image tag ubuntu domain.com/myfirstimage

# push it
docker push domain.com/myfirstimage

# pull it back
docker pull domain.com/myfirstimage
```
Voilà 
