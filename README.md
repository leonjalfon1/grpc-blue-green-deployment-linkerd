# Blue/Green Deployment for GRPC App with Linkerd
---


- In this demo we will see how to implement blue/green deployment for a gRPC application.
 - Linkerd (service mesh) is installed in the cluster and is used to manage the microservices.
 - In general what we will do is the following:
 
    1) Deploy the "v1" application to create the "production" environment (green)
    2) Use "split traffic" to manage the routing to the application and provide
    3) Deploy the "v2" application to create the new application version (blue)
    4) Allow access to the blue version by using a different url
    5) After perform the tests, route the traffic to "v2" instead of to "v1"



## Prerequisites

 - kubectl
 - aws cli
 - iam-authenticator
 - eksctl
 - grpcurl
```
go get github.com/fullstorydev/grpcurl
go install github.com/fullstorydev/grpcurl/cmd/grpcurl
export PATH=$PATH:$HOME/go/bin
```

---

## 1) Cluster Provisioning

 -  To create an empty cluster, run:
```
eksctl create cluster -f ./1-cluster-provisioning/eks-cluster.yaml
```

---

## 2) Install Linkerd

 - Install the linkerd cli
```
curl -sL https://run.linkerd.io/install | sh
```

 - add linkerd to your path with
```
export PATH=$PATH:$HOME/.linkerd2/bin
```

 - Test installation
```
linkerd version
```

 - Validate the Kubernetes cluster
```
linkerd check --pre
```

 - Install Linkerd onto the cluster
```
linkerd install | kubectl apply -f -
```

 - Validate the installation
```
linkerd check
```

---

## 3) Install nginx ingress controller

 - Install the nginx ingress controller:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.27.1/deploy/static/mandatory.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.27.1/deploy/static/provider/aws/service-l4.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.27.1/deploy/static/provider/aws/patch-configmap-l4.yaml
```

 - Test that everything is working as expected:
```
kubectl get svc -n ingress-nginx
```

 - Browse to the load balancer dns


## 4) Create alias on Route53 to access the application

 - Use an existent hosted zone (seladevops.com in my case) to create a record set with the following details:
```
Name: grpc.seladevops.com
Type: A - IPv4 address
Alias: Yes
Alias Target: <ingress load balancer dns>
Routing policy: simple
Evaluate target health: no
```

 - Test the configuration by browse to (you should receive error 404 from nginx):
```
https://grpc.seladevops.com
```

---

## 5) Create a SSL certificate for TLS termination

 - Create a self signed certificate by run (grpc.seladevops.com in my case):
```
mkdir ./2-base-application/cert
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./2-base-application/cert/tls.key -out ./2-base-application/cert/tls.crt -subj "/CN=grpc.seladevops.com/O=grpc.seladevops.com"
```

 - Create a kubernetes secret to store the certificate:
```
kubectl create secret tls grpc-certificate --key ./2-base-application/cert/tls.key --cert ./2-base-application/cert/tls.crt
```
 
---

## 6) Deploy the Base Application (gRPC)

 - Deploy the base application
```
kubectl apply -f ./2-base-application/app.yaml
```

 - Deploy a ClusterIP service to expose the application (internally)
```
kubectl apply -f ./2-base-application/svc.yaml
```

 - Deploy a ingress resources to allow reach the application
```
kubectl apply -f ./2-base-application/ingress.yaml
```

 - Wait until the application is up and running:\
```
kubectl get pods -w
```

---

## 7) Browse the application

 - Check all services provided by the gRPC server by run:
```
grpcurl -insecure grpc.seladevops.com:443 list
```

 - Check the interface for calling the helloworld.Greeter service:
```
grpcurl -insecure grpc.seladevops.com:443 list helloworld.Greeter
```

 - Query the description of specific protocol parameters of the SayHello interface:
```
grpcurl -insecure grpc.seladevops.com:443 describe helloworld.Greeter.SayHello
```

 - Call the SayHello interface and specify the name parameter
```
grpcurl -insecure -d '{"name": "gRPC"}' grpc.seladevops.com:443 helloworld.Greeter.SayHello
```
```
grpcurl -insecure -d '{"name": "World!!"}' grpc.seladevops.com:443 helloworld.Greeter.SayHello
```

---

## 8) Configure traffic split to access the application

 - Create a base service to be used by the traffic split to route the traffic:
```
```

---

## 9) Deploy new version

 - xx
```
```

---

## 10) Access the application by ingress

 - Create a self signed certificate by run (grpc.seladevops.com in my case):
```
mkdir ./5-expose-by-ingress/cert
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./5-expose-by-ingress/cert/tls.key -out ./5-expose-by-ingress/cert/tls.crt -subj "/CN=grpc-test.seladevops.com/O=grpc-test.seladevops.com"
```

 - Create a kubernetes secret to store the certificate:
```
kubectl create secret tls grpc-test-certificate --key ./5-expose-by-ingress/cert/tls.key --cert ./5-expose-by-ingress/cert/tls.crt
```

 - Use an existent hosted zone (seladevops.com in my case) to create a record set with the following details:
```
Name: grpc-test.seladevops.com
Type: A - IPv4 address
Alias: Yes
Alias Target: <ingress load balancer dns>
Routing policy: simple
Evaluate target health: no
```

 - Test the configuration by browse to (you should receive error 404 from nginx):
```
https://grpc-test.seladevops.com
```

---

## 11) Switch traffic to the new application



