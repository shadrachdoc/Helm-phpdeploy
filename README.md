# Overview
 The repository contains the code to build the application (php) as a Docker container and deploy it in a Kubernetes cluster as a load balanced service.
The application is presented as a http service. This simple php web server serves on port 80. When php web server load increase it will autoscale up to 10 pods. 


The IP to reach the website is the IP of Ingress server. Under the ingress, service has been configured using the labels to reach the corresponding pods on port 80 which was deployed using deployment.

Description of files:

dockerfile: This is the Docker configuration that can be used to build the container image for the application. This docker runs service on port 80. Since port 80 is default port.

deployment.yaml : This YAML configuration is to create Kubernetes deployment using the container image created by the dockerfile. The deployment defines a replica set of 2 pods and exposes port 80 to access the application.

service.yaml : This YAML configuration is to create Kubernetes service to load balance the requests to the pods created by the deployment.

ingress.yaml: This YAML configuration is to create Kubernetes ingress service, to get the request on port 80 of the worker node IP and pass on to the service.

Above all configured as helm3 chart 


# Assumptions
•	Using a Linux system as the host for running the minikube cluster.

# Strategy/Architecture:
                        Request --> Ingress (http://workernodeIP:80) --> K8S service --> K8S pods
In this application deployment process, Kubernetes components such as Deployment, Service, Ingress, and probes are being used.And k8s deployment with replica set more than 2 and configured under k8s services using labels given in the deployment.
Before a pod is being put in service to serve the request, readiness probe has been put in place to check whether the pod is ready to serve the request by triggering url

Liveliness probe has also been configured to check whether the pod is serving perfectly. If not, k8s will recreate the pod.

# Prerequisites:
 1) minikube  
 2) git
 3) Helm-Chart
 4) Metric server 
 5) Host entry needed in DNS server If you are testing in local then need to update in /etc/hosts with version release name and site name 

Note : I have used below command to increase to site load  (please get ip address from ingress and change it on below command)

 `$ kubectl run -i --tty load-generator1 --rm --image=busybox --restart=Never -- /bin/sh  -c "echo 192.168.15.129 php1-examplewebtest.com >> /etc/hosts; while sleep 0.01; do wget -q -O- http://php1-examplewebtest.com; done"`
 
    <releaseName>.<siteName>.com  (which we will get it from NOTE) 
 
 Example :
 
 ` root@ubuntu:~# helm install php3 ./apache/
W0119 01:37:34.214701  320736 warnings.go:70] autoscaling/v2beta1 HorizontalPodAutoscaler is deprecated in v1.22+, unavailable in v1.25+; use autoscaling/v2beta2 HorizontalPodAutoscaler
W0119 01:37:34.277410  320736 warnings.go:70] autoscaling/v2beta1 HorizontalPodAutoscaler is deprecated in v1.22+, unavailable in v1.25+; use autoscaling/v2beta2 HorizontalPodAutoscaler
NAME: php3
LAST DEPLOYED: Wed Jan 19 01:37:34 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  http://php3-examplewebtest.com`

 
 # Metric server installation procedure:
 
 Please use below command to enable metric server to monitor pods parametes 
 
 `$ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`

# Steps to deploy the application on k8s:

1)	Download the source code from repo using below command 

      `$ git clone https://github.com/shadrachdoc/Helm-phpdeploy.git`
2)	Trigger below command to check helm chat 
      `$ helm install <release name> --dry-run --debug ./apache/`
3)  Trigger helm chart to deploy the application 
      `$ helm install -f /root/apache/values.yaml <release name> apache`
         
Note : please rename <release name> with your release name  
