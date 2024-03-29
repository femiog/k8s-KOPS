# kops-kubernetes-cluster-configuration
# Landmark Technologies,  -    Landmark Technologies 
# Tel: +1 437 215 2483,   -     +1 437 215 2483 
# mylandmarktech@gaIL.com,  -    www.mylandmarktech.com 

# Setting up Kubernetes (K8s) Cluster on AWS Using KOPS

1.kops is a software use to create production ready k8s cluster in a cloud provider like AWS.

2. kOPS SUPPORTS MULTIPLE CLOUD PROVIDERS

3. Kops compete with managed kubernestes services like EKS, AKS and GKE

4. Kops is cheaper than the others.

5. Kops create production ready K8S.

6. KOPS create resources like: LoadBalancers, ASG, Launch Configuration, woker node Master node (CONTROL PLANE.

7. KOPS is IaaC

#!/bin/bash
# 1) Create Ubuntu EC2 instance in AWS

## 2a) create kops user
``` sh
 sudo adduser kops
 sudo echo "kops  ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/kops
 sudo su - kops
 ```
 ##  2a) install AWSCLI
  ```sh
 sudo apt install awscli -y 
 #or
 sudo apt update -y
 sudo apt install unzip wget -y
 sudo curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
 sudo apt install unzip python -y
 sudo unzip awscli-bundle.zip
 sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
 ```
# 3) Install kops software on ubuntu instance:

 	#Install wget if not installed
 	sudo apt install wget -y
 	sudo wget https://github.com/kubernetes/kops/releases/download/v1.22.0/kops-linux-amd64
 	sudo chmod +x kops-linux-amd64
 	sudo mv kops-linux-amd64 /usr/local/bin/kops
 
# 4) Install kubectl
```sh
 sudo curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
 sudo chmod +x ./kubectl
 sudo mv ./kubectl /usr/local/bin/kubectl
```
# 5) Create an IAM role from AWS Console or CLI with below Policies.

	AmazonEC2FullAccess 
	AmazonS3FullAccess
	IAMFullAccess 
	AmazonVPCFullAccess


Then Attach IAM role to ubuntu server from Console Select KOPS Server --> Actions --> Instance Settings --> Attach/Replace IAM Role --> Select the role which
You Created. --> Save.



# 6) create an S3 bucket. The s3 bucket will act as our key value store for etcd.
# Execute the commands below in your KOPS control Server. use unique s3 bucket name. If you get bucket name exists error.

	aws s3 mb s3://bucketog23

	aws s3 ls
	
    ex: s3://bucketog23

    #or use the command below to expose the variable
    export MY_VARIABLE=s3://bucketog23

Expose environment variable:

    # Add env variables in bashrc
    vi .bashrc
	
	# Give Unique Name And S3 Bucket which you created.
	export NAME=class.k8s.local
	export KOPS_STATE_STORE=s3://bucketog23

	
	#refresh the file on the CLI with the command below
    source .bashrc
	
# 7) Create sshkeys before creating cluster

    ssh-keygen
 

# 8) Create kubernetes cluster definitions on S3 bucket

	kops create cluster --zones us-east-2c --networking weave --master-size t2.medium --master-count 1 --node-size t2.medium --node-count=2 ${NAME}
	
	kops create secret --name ${NAME} sshpublickey admin -i ~/.ssh/id_rsa.pub
	
# The command above will copy the shh key into our KOPS. This will allow us access KOPS via ssh.

# 9) Create kubernetes cluster

	 kops update cluster ${NAME} --yes
	 
	 #The command above will create our cluster that will consist of a master node, 2 worker nodes and a VPC in the specified region.

# 10) Validate your cluster(KOPS will take some time to create cluster ,Execute below commond after 3 or 4 mins)

	   kops validate cluster
	   
	   Suggestions:
 * validate cluster: kops validate cluster --wait 10m
 * list nodes: kubectl get nodes --show-labels
 * ssh to the master: ssh -i ~/.ssh/id_rsa ubuntu@api.class.k8s.local
 * the ubuntu user is specific to Ubuntu. If not using Ubuntu please use the appropriate user based on your OS.
 * read about installing addons at: https://kops.sigs.k8s.io/operations/addons.


# 11) connect to the master node (via the KOPS server)
    
    ssh -i ~/.ssh/id_rsa ubuntu@18.223.115.43
    
    #ubuntu@kops:~$ ssh -i ~/.ssh/id_rsa ubuntu@ 172.20.55.224 (master node public IP)
    
    
    #Please note that in the above, we are using the masters node current Public IP address. Public IP address because the master node is in a different vpc than  KOPS. Hence, needs Public IP to establish connection. 
    
# 11) To list nodes

	  kubectl get nodes 
 
# 12) To Delete Cluster

   kops delete cluster --name=${NAME} --state=${KOPS_STATE_STORE} --yes  #See command below
   
    kops delete cluster --name class.k8s.local --state=s3://bucketog22 --yes

   
====================================================================================================


13 # IF you want to SSH to Kubernetes Master or Nodes Created by KOPS. You can SSH From KOPS_Server

sh -i ~/.ssh/id_rsa ubuntu@ipAddress
ssh -i ~/.ssh/id_rsa ubuntu@3.90.203.23
  
``
