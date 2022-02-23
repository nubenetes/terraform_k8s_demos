# Provisioning Kubernetes clusters on AWS with Terraform and EKS
- [Introduction](#introduction)
- [IAM Roles and policies. Prerequisites](#iam-roles-and-policies-prerequisites)
    - [IAM user and permissions](#iam-user-and-permissions)
        - [create IAM role](#create-iam-role)
        - [create API Access key/-secret](#create-api-access-key-secret)
        - [Create user "eksuser"](#create-user-eksuser)
        - [create keypair (to gain ssh access to EC2 instances)](#create-keypair-to-gain-ssh-access-to-ec2-instances)
- [Terraform and AWS credentials](#terraform-and-aws-credentials)
- [VPC Peering and Direct Connect](#vpc-peering-and-direct-connect)
- [Links of interest](#links-of-interest)

## Introduction
This repo is based on https://github.com/k-mitevski/terraform-k8s . 
**Detailed documentation can be found [here](https://learnk8s.io/terraform-eks).**

It has been tested against terraform v0.13.5 (it won't work with the latest terraform release without changes). 

This repository contains the sample code necessary to provision an EKS cluster with the ALB Ingress Controller.

Code samples:

1. [Parametrising clusters as Terraform modules](06_terraform_envs_customised/README.md)
2. [Kubernetes files to deploy an "Hello World" application](kubernetes/README.md)

## IAM Roles and policies. Prerequisites
### IAM user and permissions
Your IAM user needs to have certain privileges to e.g. create all the required resources and objects.  According AWS Best Practices you should *never* use your root account for working with AWS services. E.g. to demonstrate the Hands-On lectures, the user "eksadmin" has been used.

- Login with an admin of your AWS account: go to "IAM" => "users" => click on your user => "Permissions" => "Add permission" => then search for "IAMUserChangePassword" and attach this policy:
    - IAMUserChangePassword

There are now different 2 solutions to follow, to grant full admin access or to provide a dedicated list of policies (being the second one more restrictive an preferred):
1. IAM Solution 1. Provide Full Admin access  
Go to "Groups" within your user => Group Name: EksAdmin => "Add permission" => then search for "AdministratorAccess" and attach this policy. Basically your group (and "eksadmin" user) just requires *one* policy being attached:
    - AdministratorAccess  

2. IAM Solution 2 (alternative to previous step). Provide a dedicated list of privileges/policies  
To cover all the required privileges, first you have to create 2 additional policies:  

EKS-Admin-policy and review [this url](https://docs.aws.amazon.com/eks/latest/userguide/security_iam_id-based-policy-examples.html) if further permissions are required:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "eks:*"
            ],
            "Resource": "*"
        }
    ]
}
```

CloudFormation-Admin-policy:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudformation:*"
            ],
            "Resource": "*"
        }
    ]
}
```

Finally, assign the following policies to your IAM Group you are going to use:
  - AmazonEC2FullAccess
  - IAMFullAccess
  - AmazonVPCFullAccess
  - EKS-Admin-policy  (created above with a json file)
  - CloudFormation-Admin-policy (created above with a json file)

#### create IAM role
* open ```https://console.aws.amazon.com/iam/``` and choose _Roles_ => _create role_  
* choose _EKS_ service followed by _Allows Amazon EKS to manage your clusters on your behalf_  
* choose _Next: Permissions_
* click _Next: Review_
* enter a *unique* Role name, "EKS-role" and click *_Create Role_*

#### create keypair (to gain ssh access to EC2 instances)

* open EC2 dashboard ```https://console.aws.amazon.com/ec2```
* click _KeyPairs_ in left navigation bar under section "Network&Security"
* click _Create Key Pair_
* provide name for keypair, "eks" and click *_Create_*
* !! the keypair will be downloaded immediately => file *eks.pem* !!

#### Create user "eksuser"
* Create "eksuser" assigned to "EksUsers" Group. 
* Create "EksUsers" group with the following policies/permissions:
	* AmazonEKSClusterPolicy
    * AmazonEKSServicePolicy

#### create API Access key/-secret
* create key+secret via AWS console
  AWS-console => IAM => Users => "eksuser" => tab *Security credentials* => button *Create access key*

## Terraform and AWS credentials
This terraform template requires the permissions of the above described "eksadmin" user (inherited by the eksadmin group). Make sure your aws credentials (like i.e. 'aws configure') keep the same aws user identity during the whole EKS lifecycle with terraform (apply, update, destroy). 

## VPC Peering and Direct Connect
We have no details regarding the architecture requirements and whether other components (like RDS, DynamoDB) will be deployed within the same VPC where each EKS Cluster is running. If this is the case, subnets and security access can be automated by us via terraform. Otherwise connectivity between VPC needs to be setup via AWS PVC Peering (VPCs we might not be in charge of). 

On-premise services like Bitbucket, Nexus and Jenkins need to be connected to AWS EKS in a secure manner via AWS Direct Connect in order to avoid exposing those connections on the public internet. A simpler alternative would be to expose the connection between the on-premise CICD toolchain and EKS throughout the Internet (without a dedicated link like AWS Direct Connect). 

## Links of interest
- https://learnk8s.io/terraform-eks 
- https://github.com/aws/eks-charts 
    - [AWS Load Balancer Controller Helm Chart for Kubernetes](https://github.com/aws/eks-charts/tree/master/stable/aws-load-balancer-controller) 
- https://docs.aws.amazon.com/eks/latest/userguide/security_iam_id-based-policy-examples.html 
- https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html
