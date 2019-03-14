---
layout: post
title: Kubernetes cluster setup on AWS EC2
categories: Kubernetes cluster setup AWS EC2 cloud native microservices container management platform docker.    
description: Setting up Kubernetes cluster on AWS EC2 from scratch. After launching an EC2 in AWS, I am going to show how we can setup the Kubernetes cluster on AWS EC2.  
---

### Intro

This is a blog post about the steps I have followed to setup a Kubernetes cluster from scratch by launching new Ubuntu 18.04 AWS EC2 instances, one for master and another for worker node. In this setup the Kubernetes cluster is using Flannel CNI plugin to implement the network for communication. 

### Security Groups

The components of a Kubernetes cluster communicates through different ports and protocols, for example Kubernetes API server, etcd server, Kubelet APi, flannel CNI, etc uses different ports(6443, 10250,..) and protocols(TCP, UDP). So we should enable these ports and protocols in the security group while launching the EC2 instances. The detailed list of ports and protocols required for the master node and worker node are shown in the following pages:
1. [Kubernets required ports](https://kubernetes.io/docs/setup/independent/install-kubeadm/#check-required-ports)
2. [Flannel required ports](https://github.com/coreos/coreos-kubernetes/blob/master/Documentation/kubernetes-networking.md#port-allocation)  
&nbsp;  

### Launching Insatnces

Launch 2 Ubuntu 18.04 AWS EC2 instances with the proper security groups and IAM Roles. These 2 instances are for Kubernetes master node and worker node. Based on the requirement we have to add the permissions to the role assigned to the EC2 instances.   

### Installation on all nodes

Run the following commands in the same order in all the nodes to install docker, kubelet, kubeadm, and kubectl.

<script src="https://gist.github.com/abhidsm/3737377fc4c6946c7cb9bbbdc5c9b631.js"></script>

### Installation on master node

Run the following commands in master node to create the Kubernetes cluster, configure the kubectl and install the Flannel CNI plugin 

<script src="https://gist.github.com/abhidsm/3748048cf1074d5b5dbfe45857367507.js"></script>

### Installation on worker node 

The "kubeadm init ..." will show you a "kubeadm join ..." command at the end of the installation, which can be used to join the node to the Kubernetes cluster. The command will be something like the following:

```bash
 sudo kubeadm join $controller_private_ip:6443 --token $token --discovery-token-ca-cert-hash $hash   
```   
&nbsp;

### Conclusion

Now the Kubernetes cluster setup is done, we have one master node and one worker node running on 2 Ubuntu 18.04 EC2 instances. To test the cluster working properly, we can run the following commands:

```bash
 kubectl get nodes
 # This will print the list of nodes associated with the cluster
```  
&nbsp;
```bash
 kubectl create -f https://k8s.io/examples/admin/dns/busybox.yaml
 kubectl exec -ti busybox -- nslookup kubernetes.default
 # The first command will create a pod with busybox container running. 
 # The second command will test the DNS working properly by nslookup an internal pod name.
```

Once everything is working fine, that's it, the setup is done. Now you can deploy your application in to Kubernetes cluster.    
 
