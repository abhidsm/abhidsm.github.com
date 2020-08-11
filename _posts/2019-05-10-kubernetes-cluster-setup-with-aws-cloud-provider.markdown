---
layout: post
title: Kubernetes Cluster Setup with AWS Cloud Provider 
categories: Kubernetes Cluster Setup with AWS Cloud Provider to deploy an application with container image stored in an ECR repository. 
tags: Kubernetes Cluster Setup with AWS Cloud Provider to deploy an application with container image stored in an ECR repository. 
description: Kubernetes Cluster Setup with AWS Cloud Provider to deploy an application with container image stored in an ECR repository.
---

### Introduction

In my [previous blog post]({% post_url 2019-04-14-kubernetes-cluster-setup-on-aws-ec2 %}), I have shown you how to setup a Kubernetes cluster on AWS EC2 instances. I tried deploying a sample application with a public docker image from DockerHub and it is working fine. But when I tried to deploy an application with an image from the AWS ECR, we started getting permission errors. Even though I have given all the necessary permissions for the instance profile role, the pod couldn’t fetch the image from ECR and showing ImagePullBackOff error.

Then we found this [documentation](https://kubernetes.io/docs/concepts/containers/images/#using-aws-ec2-container-registry) from Kubernetes where they are saying “Verify kubelet is running with --cloud-provider=aws” to make it work.   
<!--more-->
### AWS Cloud Provider: 

Kubernetes Cloud Providers provide a method of provisioning cloud resources through Kubernetes via the --cloud-provider option. In AWS, this flag allows the provisioning of EBS volumes and cloud load balancers. 

Configuring a cluster for AWS requires several specific configuration parameters in the infrastructure.  

#### AWS IAM Permissions 

For the aws-cloud-controller-manager to be able to communicate to AWS APIs, you will need to create a few IAM policies for your EC2 instances. You can find the IAM policy in detail in the following link: [Click Here](https://github.com/kubernetes/cloud-provider-aws#iam-policy)

#### Infrastructure Configuration 

* Apply the roles and policies to Kubernetes masters and workers 
* Set the hostname of the EC2 instances to the private DNS hostname of the instance.  We can change the hostname to private dns name using the command "sudo hostname <Private DNS (from AWS Console)>". 
* Label the EC2 instances (tagging) with the key "kubernetes.io/cluster/<cluster-name>" and value can be <owned|shared>. For Cloud provider AWS add the tag with key "kubernetes.io/cluster/kubernetes" and value "owned" for cluster name kubernetes. 

#### Cluster Configuration 

On the Kubernetes side, we need to make sure that the --cloud-provider=aws command-line flag is present for the API server, controller manager, and every Kubelet in the cluster.  

Since we are using kubeadm to initialize the cluster and join the node, we have to create some configuration files and add the --cloud-provider=aws option to use while running the kubeadm command. 

### Kubeadm configuration changes: 

#### Kubeadm init 

The configuration file for kubeadm init is as follows: 

<script src="https://gist.github.com/abhidsm/bf11b9d0cbae5bf2e42dfff6fc5576a1.js"></script>

The command to run kubeadm init with configuration file is: “sudo kubeadm init --config kubeadm_config.yml” 

#### Kubeadm join 

The configuration file for kubeadm join is as follows: 

<script src="https://gist.github.com/abhidsm/09acc41b3d3b7aacec20f1ab057bc5b7.js"></script>

The command to run kubeadm join with configuration file is: “sudo kubeadm join --config kubeadm_join.yml” 

Now we are all set to deploy an application with an image from ECR :)


