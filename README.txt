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
			
			kubectl version --short=true
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
		docker tag imageid your-login/docker-demo
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

Access App from Outside Cluster

AWS 
	External Load balancer
Other Providers
	HAProxy, Nginx
	Expose Port directly
	
Pod yml
	app containers
Service yml
	loadbalancer
Point Hostname to LoadBalancer


L21
Demo Service with AWS ELB

kops create cluster --name=x.dprats.xyz --state=s3://my-kubernetes-learner-bucket --zones=ap-south-1a --node-count=2 --node-size=t2.micro --master-size=t2.micro --dns-zone=x.dprats.xyz
kops update cluster x.dprats.xyz --yes --state=s3://my-kubernetes-learner-bucket

git cone https://github.com/wardviaene/kubernetes-course

kubernetes-course/first-app
	
kubectl create -f first-app/helloworld.yml 
kubectl create -f first-app/helloworld-service.yml 

Route53 
	confire new record with ELB ALIAS
	
kops delete cluster --name x.dprats.xyz --state=s3://my-kubernetes-learner-bucket --yes

Vagrant  AWS 
	https://github.com/mitchellh/vagrant-aws



S2
Kubernetes BASICS

L22
Node Architecture 

Architecture Overview
Pods
	single or multiple containers
		containers run using Docker
		containers communicate
			using port numbers 
	within cluster pods can communicate but over the network
kubelet
	resposible to run pods in a node 
kube-proxy
	for a new pod, change iptable rules
		to make traffice routable with cluster 
AWS ELB 
	communicate to nodes thru IPtables 
	IPtables has rules to transfer traffic b/w nodes or pods 

	

L23
Replication controller

Scaling pods

App 
	stateless 
		horizontal scaling
traditional databases
	stateful
Web Apps
	cab be made stateless
		session management outside container
		files 2b saved can't be locally saved
			either on external storage or shared service 
			AWS S3 is helpful
	12factor.net
Stateful apps
	use Volumes
	verticall scaling
		add cpu, ram, disks etc

Replication Controller
	scaling pods 
	pod replicas
	automatic replacement of pod on fail, delte, terminate
	yaml file 
		kind: ReplicationController
		template:
			pod definition

L24
Demo: Replication Controller

For stateless apps -

minikube start
cd kubernetes-course
kubectl get node
kubectl create -f kubernetes-course/replication-controller/helloworld-repl-controller.yml 
kubectl get pods
kubectl describe pod helloworld-controller-d6j45
kubectl delete pod helloworld-controller-d6j45
kubectl scale --replicas=4 -f kubernetes-course/replication-controller/helloworld-repl-controller.yml
	or 
	kubectl scale --replicas=4 rc/helloworld-controller
kubectl get rc
kubectl delete rc/helloworld-controller
kubectl get pods


L25
Deployments

Replication Sets

Replica Set 
	Next Generation Replication Controller
	new selection feature based on filtering according to set of values
		more complex value assignment unlike ReplicationController
		
	It is used by Deployment Object than ReplicationController
	
Deployments 
	declaration
		app deployment and updates
	deployment object
		you define state of your application
			so, k8s ensure cluster macthes desired state
	Just using ReplicationController or ReplcationSet 
		very cumbersome to deploy app 
Deployment Object 
	Deployment	
		create
		update 
		rolling updates (0 downtime)
		roll back to previous version
		pause/resume (even a %)
Deployment Yaml
	Kind: Deployment 
	template:
		defines a pod 

kubectl get deployments 
kubectl get rs 
kubectl get pods --show-labels
kubectl rollout status deployment/helloworld-deployment 
kubectl set image deployment/helloworld-deployment k8s-demo:k8s-demo:2
	run another version of demo version 
kubectl edit deployment/helloworld-deployment
	similar to above and more 
kubectl rollout status deployment/helloworld-deployment 
kubectl rollout histroy deployment/helloworld-deployment
kubectl rollout undo deployment/helloworld-deployment 
	to previous version
kubectl rollout undo deployment/helloworld-deployment --to-revision=n


L26
Demo: Deployments 

cat deployment/helloworld-deployment.yml 

kubectl create -f deployment/helloworld.yml
kubectl get deployments 
kubectl get rs 
kubectl get pods --show-labels
kubectl rollout status deployment/helloworld-deployment 
kubectl expose deployment helloworld-deployment --type=NodePort
	created a service , exposed it with the port 
kubectl get service 
kubectl describe service helloworld-deployment
minikube service helloworld-deployment --url
http://192.168.99.100:30029
curl http://192.168.99.100:30029

kubectl set image deployment/helloworld-deployment k8s-demo=wardviaene/k8s-demo:2
kubectl rollout status deployment/helloworld-deployment 
	or 
	kubectl get deployments 
curl http://192.168.99.100:30029
kubectl rollout history deployment/helloworld-deployment
	while creating use 
		kubectl create -f deployment/helloworld.yml --record 
			this one gives histroy better 
kubectl rollout undo deployment/helloworld-deployment 
kubectl rollout status deployment/helloworld-deployment 
kubectl rollout history deployment/helloworld-deployment
kubectl edit deployment/helloworld-deployment
	spec:
	  replicas: 3
	  revisionHistoryLimit: 100
	  selector:
kubectl set image deployment/helloworld-deployment k8s-demo=wardviaene/k8s-demo:2
kubectl rollout history deployment/helloworld-deployment
kubectl rollout undo deployment/helloworld-deployment --to-revision=3
kubectl rollout history deployment/helloworld-deployment
curl http://192.168.99.100:30029

L27
Services

Pods
	dynamic, come and go 
		ReplicationController	
			terminated and created during scaling 
		Deployments 
			updating image version 
				terminate and new created 
	always should be connected using service 
Service 
	kubectl expose 
		to external access 
	creating service
		endpoint 
			ClusterIP - virtual IP accessed from within cluster
			NotepOrt - reachable exertnal, same on each node 
			LoadBalancer - created by cloud provider , access each node 
			Also possible to use DNS names
				ExternalName in service 
					needs DNS add-on enabled
Service Definition
	kubectl expose creates same
		default range 30000-32767
		--service-node-port-range to kube-apiserver(init script)
		ensure no collision
		use AWS ELB in front to use default http ports

		
		
L28
Demo: Services

kubectl get node 
kubectl get pods 
kubectl create -f first-app/helloworld.yml 
kubectl get pods 
kubectl describe pod nodehelloworld.example.com

cat first-app/helloworld-nodeport-service.yml
cat first-app/helloworld.yml
kubectl create -f first-app/helloworld-nodeport-service.yml
minikube service helloworld-service --url
	on AWS check services 
		kubectl get services
		goto nodes and change security groups 
curl http://192.168.99.100:31001
kubectl describe service helloworld-service
	or kubectl describe svc helloworld-service
	NodePOrt is static
	IP can be static, for that defie it in yaml 
kubectl get svc
kubectl delete svc helloworld-service



L29
Labels 

Labels 
	key/value pairs
	used to tag resources 
	label objects
		eg pods as a organizational structure etc 
	not unique
	multiple labels to a object 
	Label selectors 
		filter down on an object 
		use matching expression to match labels 
	tag nodes 
		tag node 
		use nodeSelector to run a pod on node 
	
		kubectl label nodes node1 hardware=high-spec 
		kubectl label nodes node2 hardware=high-spec 
		then 
			in a pod yaml
				use nodeSelector
					hardware: high-spec 

L30
Demo: nodeSelector Using Labels

kubectl get nodes 

cat deployment/helloworld-nodeselector.yml 

kubectl get nodes --show-labels 
kubectl create -f deployment/helloworld-nodeselector.yml 
	using nodeSelector 
	
kubectl get deployments
kubectl get pods 
	pods failed to schedule 
	check any pod 
		kubectl describe pod helloworld-deployment-4129182270-6nwj6
			it will fail to match an node with nodeSelector specified 
kubectl label nodes minikube hardware=high-spec
kubectl get nodes --show-labels 
	now wait and check 
		kubectl get pods 
		kubectl describe pod helloworld-deployment-4129182270-6nwj6
		
		
		
L31
HealthChecks
	
Application malfunctions 
	pods and containers could still be running 
	detect and resolve problems 
	types 
		1. run command in container periodically if it is working
		2. periodic check using http url app has exposed 
	application behind loadbalancer 
		to ensure availability and resiliency 
		
Health check for a container 
	yaml 
		livenessProbe: 
		
		
L32
Demo: Health Checks

cat deployment/helloworld-healthcheck.yml

kubectl create -f deployment/helloworld-healthcheck.yml

kubectl get pods 
kubectl describe pod helloworld-deployment-583969349-55zfw
	liveness
kubectl edit deployment/helloworld-deployment
	change settings 
	
	
	
L33 
Secrets 
	k8s distribute 
		credentials, keys, passwords, secret data to pods 
	k8s itself uses Secrets to provide credentials to access internal APIs 
		same mechanism can b used in ur own app 
Secrets
	native to k8s 
	other ways
		eg external vault services 
Secrets
	as env variables
	as file in a pod 
		using volumes with files 
		eg dotenv file or app can read it 
	Use external image to pull secrets(4m private repos)
	
Secrets
	generate secrets using files 
		echo -n 'root' > ./username.txt; echo -n 'password' > ./password.txt 
		kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt 
		(secret db-user-pass created)
	SSH key or SSL cert
		kubectl create secret generic ssl-certificate --from-file=ssh-privatekey=~/.ssh/id_rsa --ssl-cert-=ssl-cert=mysslcert.crt
	Secrets generation using yaml
		kind: Secret 
		echo -n 'root' | base64
		echo -n 'password' | base64
		kubectl create -f secrets-db-secret.yml
		(secret db-secret created)
		
Using Secrets
	Pod yml 
		as environment variable
			env:	
		as file 
			volumeMount:
				mountPath: 
			(<mountPath>/db-secrets/{username,password})
			
			
L34
Demo: Credentials using volumes 

cat deployment/helloworld-secrets.yml 

kubectl create -f deployment/helloworld-secrets.yml

cat deployment/helloworld-secrets-volumes.yml

kubectl create -f deployment/helloworld-secrets-volumes.yml
		
kubectl get pods 
kubectl describe pod helloworld-deployment-292348803-d677c

execute on pod 
	kubectl exec helloworld-deployment-292348803-d677c -i -t -- bash
		cat /etc/creds/username
		cat /etc/creds/password
		mount 
			tmpfs on /run/secrets/kubernetes.io/serviceaccount
			tmpfs on /etc/creds
		exit
		
		
		
L35
Demo: Running Wordpress on Kubernetes 

cat wordpress/wordpress-secrets.yml
cat wordpress/wordpress-single-deployment-no-volumes.yml
	containers 
		wordpress 
		mysql
			data is not persistent this time 
kubectl create -f wordpress/wordpress-secrets.yml
kubectl create -f wordpress/wordpress-single-deployment-no-volumes.yml
 kubectl get secrets
  kubectl get deployments
   kubectl get pods 
kubectl describe pod wordpress-deployment-2401615361-5gcbz

cat wordpress/wordpress-service.yml

kubectl create -f wordpress/wordpress-service.yml
minikube service wordpress-service --url 
	http://192.168.99.100:31001
	browse and setup wordpress admin login to dashboard 

kubectl get svc

kubectl get pods 
kubectl delete pod wordpress-deployment-2401615361-5gcbz
	data was not persistent but in the container 
		on delete it is lost 
		can be confirmed from url that again it asks to setup wordpress admin login 
	

		
L36
Kubernetes Web UI 

Web UI in place of kubectl commands

https://<kubernetes-master>/ui
	kubectl cluster-info
		Kubernetes master is running at https://192.168.99.100:8443

Manual install 
	in case not accessible or not enabled in deploy type currently 
		kubectl create -f https://rawgit.com/kubernetes/dashboard/master/src/deploy/kubernetes-dashboard.yaml 
	
	if password is asked, retrieve it using 
		kubectl config view 
		
Minikube 
	run 
		minikube dashboard --url
			http://192.168.99.100:30000
		minikube dashboard
		
L37
Demo: Web UI

http://192.168.99.100:30000



S3
Kubernetes Advanced 

L38
Service Discovery

DNS
	

			
			

		





	
		

		

		


		




	
	






	
	
	


			
	
		






		

		

		
		
		
	

	

	
	
		
	
	
	
	
	
	
	


