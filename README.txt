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
	
	minikube delete
		to delete completely from computer 
	
	minikube addons list
		minikube comes with add ons which can be enabled or disabled like dashboard 
		eg 
			minikube addons disable dashboard
			minikube addons enable heapster
		open web ui of an add on 
			minikube addons open dashboard 
	minikube dashboard 
		same as above 
	minikube dashboard --url
		to know the IP address 


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
	
Addons 
	minikube addons list
		minikube comes with add ons which can be enabled or disabled like dashboard 
		eg 
			minikube addons disable dashboard
			minikube addons enable heapster
		open web ui of an add on 
			minikube addons open dashboard 
DNS 
	> kubernetes 1.3 
		built in 
		launch auto using addon manager 
	addons 
		master node 
			/etc/kubernetes/addons
	DNS Service
		used within pods to find other services on same cluster 
	container within same pod 
		dont need DNS service
			localhost:port 
				is needed by container to connect other container 
				
DNS 
	to make DNS work 
		a pod need service definition 
			create and use service for servce discovery of a pod 
			
		command 
			services in same namespace 
				host <service-name>
					it gives IP address 
		pods and services can be launched in 2 different namespaces
			namespaces 
				logically separated different cluster 
			eg
				host <service-name>.default 
					for default namespace
			for full hostname 
				host <service-name>.<namespace>.svc.cluster.local

		host command
			for A type is to know IP address 
			for srv type is to know ports also 

kubectl run -i --tty busybox --image=busybox --restart=Never -- sh 
	Error from server (AlreadyExists): pods "busybox" already exists
		kubectl delete pod/busybox
		kubectl run -i --tty busybox --image=busybox --restart=Never -- sh
			cat /etc/resolv.conf
			exit
			
L39
Demo: Service Discovery

cat service-discovery/secrets.yml
kubectl create -f service-discovery/secrets.yml

cat service-discovery/database.yml 
kubectl create -f service-discovery/database.yml

cat service-discovery/database-service.yml
kubectl create -f service-discovery/database-service.yml

car service-discovery/helloworld-db.yml 
kubectl create -f service-discovery/helloworld-db.yml

cat service-discovery/helloworld-db-service.yml
kubectl create -f service-discovery/helloworld-db-service.yml

minikube service database-service --url
	http://192.168.99.100:32592

minikube service helloworld-db-service --url 
	http://192.168.99.100:31912
	
kubectl get pods 
kubectl logs helloworld-deployment-2141920616-2z43v

curl http://192.168.99.100:31912

kubectl get pods 
kubectl exec databse -i -t -- mysql -u root -p (rootpassword is the password 2b given on prompt)

kubectl run -i --tty busybox --image=busybox --restart=Never -- sh
	nslookup helloworld-db-service
	nslookup database-service
	telnet 192.168.99.100 31912
		GET /
	telnet helloworld-db-service 31912
		GET /

kubectl delete pod database 
kubectl delete deployment helloworld-deployment
kubectl delete svc helloworld-db-service

L40
ConfigMap
	Not secrets 
	Configuration params 
	key-value pairs
		Environment variables 
		Container cmdline arguments
		volumes 
	can contain full configuration files 
		eg.
			web server config file 
		can be mounted using volumes 
		so, config settings can be injected into containers without changing them
		
Generate COnfigMap 
	cat <<EOF > app.properties
	driver=jdbc
	database=postgres
	lookandfeel=1
	...
	EOF
	
	kubectl create configmap app-config --from-file=app.properties 
	
	use configmap
		Pod 
			As file 
			volumeMounts:
				mountPath:
			As env variables 
			env:

L41
Demo: ConfigMap

configuration for nginx
	cat configmap/reverseproxy.conf 
kubectl create configmap nginx-config --from-file=configmap/reverseproxy.conf 
(configmap "nginx-config" created)

kubectl get configmap
kubectl get configmap nginx-config -o yaml 

cat configmap/nginx.yml 
kubectl create -f configmap/nginx.yml
kubectl create -f configmap/nginx-service.yml
minikube service helloworld-nginx-service --url
	http://192.168.99.100:32526

curl http://192.168.99.100:32526 -vvvv

kubectl exec -i -t helloworld-nginx -c nginx -- bash
	ps x 
	cat /etc/nginx/conf.d/reverseproxy.conf
	
	
	
L42
Ingress Controller 

Ingress 
	>= kubernetes 1.11
	allows inbound conection
	alternative to 
		external loadbalancer
		nodeport 
	allows to 
		easily expose services to be accessible from outside the cluster 
	ingress controller 
		loadbalancer within kubernetes cluster
		there are default or you can write your own 

		rule based on 
			host 
			path 
			
Ingress 
	Kind: Ingress 
	

L43
Demo: Ingress Controller 

cat ingress/nginx-ingress-controller.yml

kubectl create -f ingress/ingress.yml 
kubectl create -f ingress/nginx-ingress-controller.yml 
kubectl create -f ingress/echoservice.yml 
kubectl create -f ingress/helloworld-v1.yml 
kubectl create -f ingress/helloworld-v2.yml 

minikube ip
curl 192.168.99.100

curl 192.168.99.100 -H 'Host: helloworld-v1.example.com'
curl 192.168.99.100 -H 'Host: helloworld-v2.example.com'

Useful to write our own loadbalancer to reduce costs 



L44
Volumes 
	store data outside containers 
	when container stops
		data in it is lost 
		stateless apps helps 
			keep data in external services 
				database, caching server, AWS S3 
	Persistent Volumes 
		can be reattached to new containers 
		
Volumes 
	Volume plugins 
		Local Volume 
			exist on a node 
			shared by containers, pods 
		AWS Cloud
			EBS
		Google CLoud
			Google disk 
		Azure 
			Azure Disk 
		Network Storage 
			NFS
			Cephfs 
Volumes 
	Stateful app 
		local filesystem 
	db like MYSQL  with persistent Volumes 
		Volumes are new since june 2016 kubernetes 
		might not be ready for production 
		be careful 
	AWS EBS 
		nodes stops working 
			pods rescheduled on another node
			volumes can be attached to new node 

		aws ec2 create-volume --size 10 --region ap-south-1 --availability-zone ap-south-1a --volume-type gp2
			Note VolumeId
		create a pod with 
			container
			volumeMounts 
			awsElasticBlockStore
		
L45
Demo: Volumes

AWS intance

aws ec2 create-volume --size 10 --region ap-south-1 --availability-zone ap-south-1a --volume-type gp2

cd kubernetes-course

kops create cluster --name=x.dprats.xyz --state=s3://my-kubernetes-learner-bucket --zones=ap-south-1a --node-count=2 --node-size=t2.micro --master-size=t2.micro --dns-zone=x.dprats.xyz
kops update cluster x.dprats.xyz --yes --state=s3://my-kubernetes-learner-bucket

vim volumes/helloworld-with-volume.yml
	awsElasticBlockStore:
		VolumeId:

kubectl get node
		
kubectl create -f volumes/helloworld-with-volume.yml 
(deployment "helloworld-deployment" created)

kubectl get pod 
kubectl describe pod <pod-name>

kubectl exec <pod-name> -i -t -- bash 
	ls -ahl /myvol/

	echo 'test1' > /myvol/myvol.txt
	echo 'test2' > /test.txt 
	
	ls -ahl /myvol/myvol.txt /test.txt 
	
kubectl get node 
kubectl describe pod <pod-name>

drain the node for the pod removal
	kubectl drain <node-name> --force 
	(node <node-name> cordones)

kubectl get pod 
kubectl describe pod <new-pod-name>
	check instance name or new node 
	check the EBS volume 

kubectl exec <new-pod-name> -i -t -- bash 
	ls -ahl /myvol/

	ls -ahl /myvol/myvol.txt 
	
	ls -ahl /test.txt 	
	ls: /test.txt: No such file or directory
	(test.txt was on the old pod deleted one, now lost)
	
Finally detach and remove EBS volume 
	delete pod 
		kubectl delete -f volumes/helloworld-with-volume.yml 
	delete EBS volume 
		aws ec2 delete-volume --volume-id VolumeId 
	
kops delete cluster --name x.dprats.xyz --state=s3://my-kubernetes-learner-bucket --yes




L46
Volume AutoProvisioning

	kubernetes plugins capability 
		provision storage for you
		
		AWS Volume Plugin
			creates Volumes in EBS before attaching to a node 
			
			done using StorageClass 
			maybe in beta version , check 
				https://kubernetes.io/docs/user-guide/persistent-volumes/
			
	auto provisioning volume 
		storage.yml (beta)
			kind: StorageClass 
			provisioner: kubernetes.io/aws-ebs 
			type: gp2 
			zone: 
			
		persisten volume claim and specify size (stable api)
		my-persistent-volumes.yml 
			Kind: PersistentVolumeClaim
			
		launch pod using the volume 
			my-pod.yml 
				volumeMounts:
				volumes:
					persistentVolumeClaim:
						claimName:
						

						
L47
Demo: WOrdpress with Volumes 

AWS instance 

cd kubernetes-course

cd wordpress-volumes 
cat storage.yml 
cat pv-claim.yml 
cat wordpress-db.yml 
cat wordpress-db-service.yml 
cat wordpress-secrets.yml
cat wordpress-web.yml
cat wordpress-web-service.yml

aws efs create-file-system --creation-token 2

to know subnet-ids and security-group ID
	aws ec2 describe-instances

aws efs create-mount-target --file-system-id <FileSystemId from aws efs create-file-system> --subnet-id <same as in the cluster> --security-groups <sgId>

vim wordpress-web.yml 
	FileSystemId
		volumes:
			nfs:
				server:
 
kubectl create -f storage.yml 
kubectl create -f pv-claim.yml 
kubectl create -f worpress-secrets.yml
kubectl create -f wordpress-db.yml 
kubectl create -f wordpress-db-service.yml

check volumes created 
	kubectl get pvc 

kubectl get pod 
kubectl describe pod <pod-name>

kubectl create -f wordpress-web.yml 
kubectl create -f wordpress-web-service.yml

Route53 
	create new record set 
		ALIAS ELB 
		
browse above url set 
	wordpress admin login setup will be first step 
	MYSQL persistent volume created on EBS block storage 
	wordpress file/media upload path on EFS 
		This path will not allow to upload ie write anything,
			bcoz the wordpress kubernetes image was not built to write on EFS 
			solution 
				1. built new image 
				2. change from cmdline 
					kubectl edit deploy/wordpress-deployment 
					add to -
					containers:
						- command:
							- bash
							- -c
							- chown www-data:www-data /var/www/html/wp-content/uploads && docker-entrypoint.sh apache2-foreground
							(change www-data to proper permissions, docker-entrypoint.sh can be found on docker Hub)
							
Volume Verification
	kubectl get pod
	kubectl delete pod <db-pod-name>
	kubectl delete pod <wordpress-pod-name-1>
	kubectl delete pod <wordpress-pod-name-2>
	kubectl get pod 
	kubectl logs <new-wordpress-pod-name-1>
	kubectl exec <new-wordpress-pod-name-1> -i -t -- bash
		ls -ahl /var/www/html/wp-content/uploads/2017/8
		(we still have the image even after pods recreated)
		
	kubectl get pod 
	kubectl logs <new-db-pod-name>
	

Deletions
	Delete EFS file system 
	Delete cluster 
		kops delete cluster --name x.dprats.xyz --state=s3://my-kubernetes-learner-bucket --yes
		
		
		
L48
Pet Sets
	>= kubernetes 1.3
	stateful applications that need 
		stable pod hostname instead of pod name with randomstring
			podname with indexes 
		multiple pod with volume 
			delteing pod not deletes volume 
	allow stateful app to use DNS to find peers
		cassandra/ElasticSearch clusters
	1 running node of Pet Set in a Pet ( a node in cassandra)
	using pet sets 
		run 5 cassandra nodes 
			cassandra-1 - cassandra-5
	dynamic names f pod can not be used in configuration files 
		as names will change 
		Pet Sets provide predictable names 
		
Pet Sets 
	order startup or teardown
	know in advance which Pet will go down 
	useful to drain data from node before shut down 
	
Pet Sets 
	currently in alpha 
	be careful 
	

	
L49
Daemon Sets
	Ensures every single node on k8s cluster runs same pod resource 
		useful to ensure a certain pod running on every node 
	pod addition or removal 
		syncronous with node 
	Usage 
		Logging
		monitoring
		Load Balancers/ Reverse Proxies/API Gateways 
		running daemon that needs one instance per physical instance 
	still api in beta 
	Kind: DaemonSet
	

L50
Resource Usage Monitoring 
	Heapster 
		enable container cluster monitoring 
		Performance analysis
		monitoring platform in k8s 
		prerequisite for pod autoscaling in k8s 
		exports cluster metrics with REST endpoints 
		different backends with heapster
			backend 
				where metric data stored 
			InfluxDb
			Google Cloud Monitoring/Logging
			Kafka 
		visualization graph 
			using grafana
			using kubernetes dashboard 
		All - Heapster, InfluxDb, Grafana 
			can be started in pods 
		yaml files of Heapster
			on github repo 
				github.com/kubernetes/heapster/tree/master/deploy/kube-config/influxdb
			on download 
				deploy whole platform using 
					addon system 
					or 
					kubectl create -f directory-with-yaml-files/
		kubelet has feature 
			cAdvisor 
				it can be separate in future 
			cAdvisor
				it will send metrics to Heapster
					Heapster will save it to influxdb
						Grafana can be used to get data from influxdb
							to visualize in graphs 


L51
Demo: resource Usage Monitoring

local or AWS -
	clone or download heapster from 
		git clone https://github.com/kubernetes/heapster.git 
	
cd heapster/
ls -ahl deploy/kube-config/influxdb/

files can be used for addons 

but another way 
	vim deploy/kube-config/influxdb/grafana.yaml
		edit where 
			Kind: Service 
	vim deploy/kube-config/influxdb/grafana.yaml
		edit where 
			Kind: Service
	vim deploy/kube-config/influxdb/influxdb.yaml
	
run all files in directory 
	kubectl create -f heapster/deploy/kube-config/influxdb/
	
	to see these services working 
		cd kubernetes-course
		kubectl create -f deployment/helloworld.yml 
		
		kubectl get svc --namespace kube-system 
		kubectl get pod 
		kubectl config view
		
		minikube service monitoring-grafana --namespace kube-system --url
		
Visualize
		browse Grafana url 
			admin/admin 
		browse dashboard url 
			minikube dashboard --url 
		

L52
Autoscaling 

Horizontoal Pod Autoscaling 
	automatic scale pod 
		using metrics 
	k8s automatic scale deployment / ReplicationController / ReplcationSet
	>= kubernetes 1.3
		autoscaling based on CPU usage metric 
		with alpha support 
			to custom metric scaling 
			env variable ENABLE_CUSTOM_METRICS=True 
				on cluster start 
				for application based metrics 
					eg - request per second etc 
	autoscaling 
		periodically query the utilization for the targeted pods 
			default 30s 
			changed using 
				--horizontal-pod-autoscaler-sync-period 
					in init-scripts when lauching the controller-manager on master node 
		will use Heapster
			to gather metrics and make scaling decisions 
			Heapster must be installed and running before autoscaling works 
		eg -
			deployment pods with CPU resource request 200m 
				200m = 200millicores or millicpu
			200m = 0.2 = 20% of a single core of the running node 
				if node has 2 cores, it is still 20% of a single core 
			auto-scaling 50% of CPU usage(50% of 200m = 100m)
			horizontal auto-scaling will increase/decrease pods to maintain a target CPU utilization of 50% (100m/10% of a core within this pod)
			
Autoscaling 
	deployment 
		resources:
			requests:
				cpu: 200m 
			
	autoscaling specification
		Kind: HorizontalPodAutoscaler
		minReplicas:
		maxReplicas:
		targetCPUUtilization: 50
	
	
L53
Demo: Autoscaling 

cd kubernetes-course/

cat hpa-example.yml 
kubectl create -f autoscaling/hpa-example.yml 

kubectl get hpa 

to generate some cpu load
	kubectl run -i --tty load-generator --image=busybox /bin/sh 
		wget http://hpa-example.default.svc.cluster.local:31001
		cat index.html 
		rm index.html 
		while true; do wget -q -O- http://hpa-example.default.svc.cluster.local:31001; done
		
	kubectl get pod 
	kubectl get hpa 
		keep checking TARGET %
			while above loop keeps running 
	
kubectl delete deployments --all	
 kubectl delete rc --all
 kubectl delete hpa --all
 kubectl delete svc --all
 kubectl delete ingress --all
 kubectl delete pods --all
 
			
			
L54
Kubernetes Administration


	



		
	
								
							
	

	




	


