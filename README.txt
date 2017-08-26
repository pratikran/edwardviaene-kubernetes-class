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

L13
Building Docker Containers

download Docker Engine

Or use 
vagrant 
	git clone https://github.com/wardviaene/devops-box.git
	
	cd devops-box/
        vagrant up
		
dockerfile
docker-demo app 
 
docker build

L14
Demo: Building docker containers

vagrant up
vagrant ssh 
cd /

sudo apt-get install docker.io 
docker --version
	1.12.6
git clone https://github.com/wardviaene/docker-demo
cd docker-demo
cat Dockerfile
add user 'ubuntu' to group 'docker'
	sudo usermod -G docker ubuntu 
	docker build .
	
docker run -p 3000:3000 -it 66298f0db7b0

curl localhost:3000

L15
Docker Image Registry

docker run 
	local development
for kubernetes
	push image to docker registry
		docker hub
		(create account on docker hub)
		docker login
		docker tag iageid your-login/docker-demo
			(docker-demo will be the ultimate image name)
		docker push your-login/docker-demo
		
		or
		during local build time 
			docker build -t your-login/docker-demo
		docker push your-login/docker-demo
			(push to docker hub or any registry to use, AWS has docker ecr)
			
run one process in one container 
	all data in container is not preserved
		use volumes to preserve data
follow 12factor.net
	how to write stateless applications

dofferent image on docker hub 

L16
Pushing docker image 
	(docker hub)
	
https://hub.docker.com
	create new account id 
	create repository
	(limited private repo but unlimited public)

docker login 
	username/password
	
docker images

docker tag 66298f0db7b0 pratikran/k8s-demo
	docker tag 66298f0db7b0 pratikran/k8s-demo:latest
	docker tag 66298f0db7b0 pratikran/k8s-demo:<version>
	
docker push pratikran/k8s-demo
	(The push refers to a repository [docker.io/pratikran/k8s-demo])
	(this will create a new repo on hub or update one)
	
L17
Runnign First APp on Kubernetes 

before container 
	create a pod
		pod describes application on kubernetes 
		pod can contain 1 or more tightly coupled containers that make the app
		those app can easily comunicate using local port numbers
		
	this app has only 1 container 

	create a file pod-helloworld.yml
	
create pod
	kubectl create -f k8s-demo/pod-helloworld.yml

L18
Demo: Running first app on kubernetes	

git clone https://github.com/wardviaene/kubernetes-course

cd kubernetes-course
(minikube would be running)
kubectl get node 
kubectl create -f first-app/helloworld.yml 
kubectl get pod 
kubectl describe pod nodehelloworld.example.com
kubectl port-forward nodehelloworld.example.com 8081:3000 
kubectl expose pod nodehelloworld.example.com --type=NodePort --name nodehelloworld-service 
	(service "nodehelloworld-service" exposed)
kubectl get service 
minikube service nodehelloworld-service --url 

L19
Demo: Useful Commands

kubectl attach nodehelloworld.example.com
kubectl exec nodehelloworld.example.com -- ls -1
kubectl exec nodehelloworld.example.com -- touch test.txt 
kubectl exec nodehelloworld.example.com -- ls -1
	(if container is killed test.txt file will disappear, volumes r used for persistency)
	

kubectl get service 
kubectl describe service nodehelloworld-service
kubectl run -i --tty busybox --image=busybox --restart=Never -- sh

L20
Service with loadbalancer on AWS 


	







	
	
	


			
	
		






		

		

		
		
		
	

	

	
	
		
	
	
	
	
	
	
	


