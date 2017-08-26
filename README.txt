The Complete Kubernetes Course

mkdir kubernetes
cd kubernetes

git init

S0
L3
Kubernetes Procedure Document

S1
L4
What is Kubernetes 

L5
Container Introduction

L6
Kubernetes setup

L7
Local Setup Minikube 

Minikube - sinlge node kubernetes cluster locally 
	download 
		https://storage.googleapis.com/minikube/releases/latest/minikube-windows-amd64.exe
		mv minikube-windows-amd64 minikube.exe 
		add to path

VirtualBox
https://github.com/kubernetes/minikube
powershell/console - 
	minikube start
	(~/.kube/config)

L8
Demo - Minikube

powershell/console - 
	minikube start
	(~/.kube/config)
	
Download -
	kubectl
		https://kubernetes.io/docs/tasks/kubectl/install/
		add to path 
	(uses confg file)
	
	kubectl run hello-minikube --image=gcr.io/google_containers/echoserver:1.4 --port=8080
	(deployment "hello-minikube" created)
	kubectl expose deployment hello-minikube --type=NodePort
	(service "hello-minikube" exposed)

	kubectl get pod
		# To check whether the pod is up and running
	
	minikube service hello-minikube --url
	(<URL>)
	
	browse <URL>	
	
	kubectl cluster-info
	
	minikube stop
	
L9
Introduction of KOPS
KOPS 
	(Kubernetes Operations)
	(for AWS)
	(Kube-up.sh lagacy tool, deprecated, no production ready environment)
	(MAC or LINUX)
	(Windows - VirtualBox or other VM)
	
Windows - 
	vangrant init ubuntu/xenial64
	vagrant up
	
L10
DEMO: KOPS install

KOPS 
	Linux or MAC
	Windows - VirtualBox or Vagrant 
	
	mkdir ubuntu && cd ubuntu
	vangrant init ubuntu/xenial64
	vagrant up
	vagrant ssh-config 
		IdentityFile(private_key), puttygen
		putty into VM
	vargrant ssh
		needs openssh client
		
L11
Demo: Prepare AWS for KOPS Install

AWS instance ubuntu/xenial64

https://github.com/kubernetes/kops
Download
	wget kops-linux-amd64
	sudo chmod +x kops-linux-amd64
	sudo mv kops-linux-amd64 /usr/local/bin/kops
	sudo apt-get install python-pip
		(to install aws cli the latest ones)
	sudo pip install awscli
	aws
	aws console - IAM
		create new user kops
			get access/secret keys
			attach policy - admin access 
	aws configure
	ls -lha ~/.aws/
	
	goto AWS S3
		new bucket
		choose region as in where cluster 2b created 
	goto AWS Route53
		create hosted zone
			use a free domain or subdomain
		configure subdomain or the domain on your domain seller site like godaddy

L12
Demo: Cluster Setup on AWS using kops

kubectl install
	wget kubectl 
		or
	curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
	sudo mv kubectl /usr/local/bin/
	sudo chmod +x /usr/local/bin/kubectl
	
for clusters ssh keys 
	ssh-keygen -f .ssh/id_rsa
	
	kops create cluster --name=x.dprats.xyz --state=s3://my-kubernetes-learner-bucket --zones=ap-south-1a --node-count=2 --node-size=t2.micro --master-size=t2.micro --dns-zone=x.dprats.xyz
	
	kops edit cluster x.dprats.xyz --state=s3://my-kubernetes-learner-bucket
	
	launch cluster 
		kops update cluster x.dprats.xyz --yes --state=s3://my-kubernetes-learner-bucket
		(see kops1 log)
		
		less  ~/.kube/config
		
		* validate cluster: kops validate cluster
		* list nodes: kubectl get nodes --show-labels
		* ssh to the master: ssh -i ~/.ssh/id_rsa admin@api.x.dprats.xyz
		The admin user is specific to Debian. If not using Debian please use the appropriate user based on your OS.
		* read about installing addons: https://github.com/kubernetes/kops/blob/master/docs/addons.md
		
		kops validate cluster --state=s3://my-kubernetes-learner-bucket
		
		kubectl get node 
			didnt work
		kubectl get nodes --show-labels 
			didnt work 
		
		kubectl run hello-minikube --image=gcr.io/google_containers/echoserver:1.4 --port=8080
			did not work
			
			error: group map [ ....... ] is already registered
			
			kubectl version
			(update kubectl to latest)
			
		kubectl get node 
		kubectl get nodes --show-labels
		kubectl run hello-minikube --image=gcr.io/google_containers/echoserver:1.4 --port=8080
		(deployment "hello-minikube" created)

		kubectl expose deployment hello-minikube --type=NodePort
		(service "hello-minikube" exposed)
		
		kubectl get service
			NAME             CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
			hello-minikube   100.64.47.98   <nodes>       8080:30943/TCP   51s
			kubernetes       100.64.0.1     <none>        443/TCP          34m
					
			add to master node security group TCP port 30943 open to all
			
		Ec2 Instance?Route53 Hosted Zone
			FInd IP(or route53 url) of Master node 
			
		Browse 
			http://IP:30943/
			
		Delete Everything
			kops delete cluster --name x.dprats.xyz --state=s3://my-kubernetes-learner-bucket
			(kops2 delete log)
			
			kops delete cluster --name x.dprats.xyz --state=s3://my-kubernetes-learner-bucket --yes
		
		
		
		
	

	

	
	
		
	
	
	
	
	
	
	


