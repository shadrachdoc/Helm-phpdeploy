# Overview
 This repository is to deploy php application in a Kubernetes cluster as a load balanced service.
The application is presented as a http service. This simple php web server serves on port 80. When there is a increase in nomber of request K8S will autoscale up to 10 pods. 


The IP to reach the website is the IP of Ingress server. Under the ingress, service has been configured using the labels to reach the corresponding pods on port 80 which was deployed using deployment.

Description of files:

deployment.yaml : This YAML configuration is to create Kubernetes deployment using the php:apache container image. The deployment defines a replica set of 2 pods and exposes port 80 to access the application.

service.yaml : This YAML configuration is to create Kubernetes service to load balance the requests to the pods created by the deployment.

ingress.yaml: This YAML configuration is to create Kubernetes ingress service, to get the request on port 80 of the ingress IP and pass on to the service.


All the configurations are configured in helm chart.



# Assumptions
•	Using a Linux system as the host for running the minikube cluster.

# Strategy/Architecture:
                        Request --> Ingress (http://ingressIP:80) --> K8S service --> K8S pods
In this application deployment process, Kubernetes components such as Deployment, Service, Ingress, and configMap are being used.And k8s deployment with replica set 1 and configured under k8s services using labels given in the deployment.


# Prerequisites:
 1) minikube  
 2) git
 3) Helm-Chart
 4) Metric server 
 5) Host entry needed in DNS server If you are testing in localhost then need to update in /etc/hosts with release name and site name 

   For Example : If ingress ip is 192.168.15.129  (to get ingress ip: Kubectl get ingress) and release name is php1, siteName (values.yaml) is examplewebtest.com  then below command needs to be executed in the localhost 
   echo 192.168.15.129 php1-examplewebtest.com >> /etc/hosts

Note : I have used below command to increase to site load  (please get ip address from ingress and change it on below command)

 `$ kubectl run -i --tty load-generator1 --rm --image=busybox --restart=Never -- /bin/sh  -c "echo 192.168.15.129 php1-examplewebtest.com >> /etc/hosts; while sleep 0.01; do wget -q -O- http://php1-examplewebtest.com; done"`
 
    <releaseName>.<siteName>.com  (which we will get it from NOTE) 
 
  
 # Metric server installation procedure:
 
 Please use below command to enable metric server to monitor pods parametes 
 
 `$ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`

# Steps to deploy the application on k8s:

1)	Download the source code from repo using below command 

      `$ git clone https://github.com/shadrachdoc/Helm-phpdeploy.git`
2)	Trigger below command to check helm chat 
      `$ helm install <release name> --dry-run --debug ./apache/`
3)  Trigger helm chart to deploy the application 
      `$ helm install <release name> ./<directory>/`
         
Note : please rename <release name> with your release name  
