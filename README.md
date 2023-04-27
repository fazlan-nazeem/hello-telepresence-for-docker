# Provision an EKS cluster with Terraform to demonstrate Telepresence for Docker

### Demo Setup

### AWS

1. AWS account
2. AWS IAM user - Note this user will need some IAM permissions including roles, policies, OpenID

### Install the AWS CLI

1. [Install](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
2. [Configure](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)

```
% aws --version
aws-cli/2.11.5 Python/3.11.2 Darwin/22.3.0 exe/x86_64 prompt/off
```

```
% aws sts get-caller-identity
{
    "UserId": "YYA4ZRSAMPLE6IDAGV2YP",
    "Account": "378929636692",
    "Arn": "arn:aws:iam::378929636692:user/dockerbisson"
}
```

### Tools to install on new laptop
1. Install brew: /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
2. Install helm: 'brew install helm'
3. Install jq: 'brew install jq'

### Terraform Cloud

1. [Create account](https://app.terraform.io/signup/account)
2. Create an org
3. [Terraform variables with AWS credentials](https://developer.hashicorp.com/terraform/tutorials/cloud-get-started/cloud-create-variable-set)

### Terraform local

From the top level of the repo

1. `brew install terraform`
2. `terraform init`
3. `terraform plan`
4. `time terraform apply -auto-approve`

### kubectl config

Get `kubectl` config ([more details](https://repost.aws/knowledge-center/eks-cluster-connection)):

```
aws eks --region us-west-2 update-kubeconfig --name demo-eks
```

The `region` and `name` values match those in the `main.tf` file.

### Deploying the app

Install the helm chart

```
cd helm
helm install --create-namespace -n hello-telepresence-for-docker app ./hello-telepresence-for-docker --debug
```

### Get all details in that namespace

```
kubectl get all -n hello-telepresence-for-docker
```

### Confirm the app is running in EKS (may take a minute or two for the app to be running)

```
open "http://$(kubectl get service -n hello-telepresence-for-docker -o json | jq -r '.items[].status.loadBalancer.ingress[].hostname')"
```

### Browser setup (optional, can use preview URL instead)
1. Install the mod header extension for chrome

### Telepresence extension setup
1. Open Docker Desktop
2. Install telepresence extension


### Demo execution

### Update the code for the app to show a new message and build the image
1. cd to the hello-app directory
2. Update the Docker file - change the ENV MESSAGE to your message and save
3. Type 'docker build -t hello-app .'

### IMPORTANT - the GUI and CLI options are separate.  You can do both, but you need to make sure the intercept is down before switching between them

### Docker desktop and telepresence setup (GUI option only)

1. Open Docker Desktop
2. Go to the telepresence extension
3. Select "Get Started" - the Select Cluster for Telepresence Connection Dialog will appear
4. The kubeconfig context should show the aws cluster
5. Select "Install telepresence on this cluster" only if this is the first connection since building the cluster
6. Select connect

### Create the intercept (GUI option only)
1. Click start intercept on the telepresence extension if you are not already on it
2. Select the "hello-telepresence-for-docker" namespace
3. The hello-telepresence-for-docker-app service should be listed
4. Click Intercept
5. A dialog appears for the Intercept
6. Make sure the target docker image is hello-app:latest
7. Target Port Number is 8080
8. Leave the other settings
9. Click Create Intercept

### Telepresence connect and creating the intercept (CLI option)

1. Type 'telepresence intercept --docker hello-telepresence-for-docker-app --namespace hello-telepresence-for-docker --port 8080:80 --docker-run -- -it --rm hello-app:latest'

### Show the container running(optional)
1. Go to the containers tab
2. Show there is a "tp-hello-telepresence-for-docker-app" container running
3. Explain that the telepresence extension runs this container when the intercept starts based on the image you specify

### Showing the intercept
Option 1: HTTP headers
1. Copy the request header string shown in the current running intercepts (or on the terminal page)
2. Put the x-telepresence-intercept-id in the name field in mod header
3. Put the rest (minus the colon after id) into the value field and make sure the green check is there
4. Refresh the hello world page to see your new message
5. Turn off the header and show the page goes back to hello world

Option 2: Preview URL
1. Click the preview URL link (or copy from the terminal page) and show the updated page
2. Refresh the original page to show it still has hello world


### Demo cleanup

### Stop the intercept (GUI)
1. Click stop intercept
2. (optional) show the local container has stopped as well

### Stop the intercept (CLI)
1. type 'telepresence leave'

### Change the app back

1. cd to the hello-app directory
2. Update the Docker file - change the ENV MESSAGE to your message and save
3. Type 'docker build -t hello-app ."



### Deleting the app from Kubernetes

```
helm delete -n hello-telepresence-for-docker $(helm ls --namespace hello-telepresence-for-docker --short)
```
### Verify the app is deleted
kubectl get all -n hello-telepresence-for-docker

### Change the kubectl context back to Docker desktop

1. type 'kubectl config use-context docker-desktop'

### Deleting the infrastructure from AWS

From the top-level of the repo

1. `terraform destroy -auto-approve`
2. Manually delete any load balancers associated with the cluster [in the web console](https://us-west-2.console.aws.amazon.com/ec2/home?region=us-west-2#LoadBalancers)
3. Manually delete the VPC associated with the cluster [in the web console](https://us-west-2.console.aws.amazon.com/vpc/home?region=us-west-2)
4. `terraform destroy -auto-approve` again

### Clearing DNS cache

If needed...

```
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder
```

### History

- Originally created as a fork of https://github.com/hashicorp/learn-terraform-provision-eks-cluster, based on a Terraform EKS tutorial: https://developer.hashicorp.com/terraform/tutorials/kubernetes/eks
- the "hello-app" is derived from https://github.com/paulbouwer/hello-kubernetes
