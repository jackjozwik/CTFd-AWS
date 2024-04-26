## Basic Setup
### Install Docker

```

sudo apt-get update
sudo apt-get install ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo apt install docker-compose
```

### Install GIT

```
sudo apt install git
```


### CTFd (Local)

```
git clone https://github.com/CTFd/CTFd.git
docker-compose up
```

access CTFd at [http://localhost:8000](http://localhost:8000/)


### Install AWS CLI (Windows)

```
Install:
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi

aws configure
```

### Install Terraform (Windows)

```
Download Terraform:

https://releases.hashicorp.com/terraform/1.8.2/terraform_1.8.2_windows_amd64.zip

setx PATH "%PATH%;C:\path\to\terraform"
```

### CLI Config

AWS CLI
```
aws configure <ACCESS_KEY> <SECRET_KEY> <REGION>
```

Docker CLI
```
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-st
din <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
```


### Docker to ECR

```
Connect to ECR (uses setup env vars):

aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <YOUR_AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com

aws ecr create-repository --repository-name my-repo

cd <DIRECTORY>
docker build -t ctfd .

docker tag ctfd:latest <YOUR_AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/ctfd-ecr-repo:latest

docker push <YOUR_AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/ctfd-ecr-repo:latest

Ex.
Web Challenge 1:

aws ecr create-repository --repository-name ctf_promptinjection_1
docker build -t ctf_promptinjection_1 .
docker tag ctf_promptinjection_1:latest <YOUR_AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/ctf_promptinjection_1:latest
docker run -p 5000:80 ctf_promptinjection_1
docker push <YOUR_AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/ctf_promptinjection_1:latest

docker build -t meltdown .
docker tag meltdown:latest <YOUR_AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/meltdown:latest
docker push <YOUR_AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/meltdown:latest
```


### Terraform

`terraform init`
`terraform plan`
`terraform apply`


### Gitlab CI/CD

#### Normal commit:
```
git add .
git commit -m ""
git push origin main
```
#### Push from github to gitlab
```
git push gitlab main --force
```



## General Knowledge
## Terraform

```
resource "<provider>_<resource_type>" "name" {
	config options...
	key1 = "value1"
	key2 = "value2"
}
```


The main terraform file for the current directory
`main.tf`

The variable declaration - think of it as setting up a form
`variables.tf`

The variable assignment - think of it as filling out that form
`terraform.tfvars`

## Environment Variables

Variables can be setup either through AWS Parameter Store or your system environment variables. Access keys **MUST** be done using your system environment variables as this grants access to further resources, think of this as your login credentials like username and password.

### Linux

Add these lines to your shell profile file (`~/.bashrc` for Bash, `~/.zshrc` for Zsh, etc.) if you want them to be set every time you open a new terminal session.

```
export AWS_ACCESS_KEY_ID="your-access-key-id"
export AWS_SECRET_ACCESS_KEY="your-secret-access-key"
export AWS_DEFAULT_REGION="us-east-1"
```

### For Windows:

You can set environment variables either through the command prompt or through the System Properties window.

```
setx AWS_ACCESS_KEY_ID "your-access-key-id"
...
```



### DNS

It is generally more appropriate to use a **CNAME record** rather than an A record. AWS Elastic Load Balancers (ELBs) can change their IP addresses over time. A CNAME record allows you to point your domain or subdomain to the ELB's DNS name, which automatically handles these IP address changes.

1. **Request the Certificate in ACM**: Terraform will start this process.
   - NOTE: Terraform may error out saying "UnsupportedCertificate: The certificate must have a fully-qualified domain name, a supported signature, and a supported key size". Continue to step 2 and re-run terraform after this entire process.
2. **Retrieve Validation CNAME Record**: After applying your Terraform configuration, go to the AWS ACM console to find the DNS validation CNAME record(s) provided by ACM.
3. **Add CNAME Record in Cloudflare**:
    - Log in to your Cloudflare account.
    - Navigate to the DNS settings for `issessions.ca`.
    - Create the CNAME records provided by ACM.
4. **Certificate Validation Process**:
    - ACM will periodically check for the presence of these DNS records.
    - Once ACM can verify the records, it will issue the certificate.

If you are creating a subdomain like `ctf.issessions.ca` from ACM, the CNAME record in Cloudflare would be:

- **Type**: CNAME
- **Name**: `_260acca77919476a83bdfb16912676d6.ctf.issessions.ca`
- **Target**: `_013f935e8836427ae0e6d25a68fcc858.mhbtsbpdnt.acm-validations.aws`


#### AWS
![image](https://github.com/jackjozwik/CTFd-AWS/assets/56364185/67cd10be-842a-4d52-9625-7e38fa02ac73)

#### Cloudflare

##### HTTP *(For Testing)*
![image](https://github.com/jackjozwik/CTFd-AWS/assets/56364185/7b0d4b4f-e7bb-4401-beec-50942a135d45)

![image](https://github.com/jackjozwik/CTFd-AWS/assets/56364185/a9e5bf63-faee-41f8-90ac-2c3200841fa8)


##### HTTPS
![image](https://github.com/jackjozwik/CTFd-AWS/assets/56364185/474aa71e-70f6-4df0-9d7b-e3d20c43f87e)

![image](https://github.com/jackjozwik/CTFd-AWS/assets/56364185/995bfc18-1ac0-46ec-bfe9-836db4c2e634)


#### Web & PWN Challenges *(Sub-sub Domain)*

Entry Name:
subdomain2.subdomain1

ex.
athena.ctf

*expands to:*

athena.ctf.issessions.ca

![image](https://github.com/jackjozwik/CTFd-AWS/assets/56364185/9ac84ca9-6334-4b77-b4e4-bb9ddfb18856)

Make note that PWN challenges will have a different Load Balancer (NLB) then the Web Challenges and main CTFd application (ALB)

![image](https://github.com/jackjozwik/CTFd-AWS/assets/56364185/64a2a8bb-ddd6-4292-8713-e990ec5e1a7d)


#### Email Server Setup (CTFd - smtp)
![image](https://github.com/jackjozwik/CTFd-AWS/assets/56364185/065c9804-37cf-43b8-ae54-567f7a06eeff)
