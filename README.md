# Provision an EKS demo cluster with Terraform

Instructions: https://developer.hashicorp.com/terraform/tutorials/kubernetes/eks

### AWS

1. AWS account
2. AWS IAM user

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
helm install -n hello-telepresence-for-docker app ./hello-telepresence-for-docker --debug
```

Get all details in that namespace

```
kubectl get all -n hello-telepresence-for-docker
```

Open the app

```
open "http://$(kubectl get service -n hello-telepresence-for-docker -o json | jq -r '.items[].status.loadBalancer.ingress[].hostname')"
```

### Deleting resources / cleanup

From the top-level of the repo

1. `terraform destroy -auto-approve`
2. Manually delete any load balancers associated with the cluster [in the web console](https://us-west-2.console.aws.amazon.com/ec2/home?region=us-west-2#LoadBalancers)
3. `terraform destroy -auto-approve` again
