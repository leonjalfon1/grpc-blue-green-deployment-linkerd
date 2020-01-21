# Blue/Green Deployment for gRPC Application with Linkerd
---

 ## Tasks

 1) Deploy the application "v1" (before blue/green deployment)
 2) Allow access to the application trough the ingress controller
 3) Configure TrafficSplit to manage the routing to the application (green version)
 4) Deploy a new version of the application "v2" (blue version)
 5) Allow access to the application trough the ingress controller
 6) Switch traffic to the new version
 7) Rollback the deployment

---

## 1) Deploy the base application (before blue/green deployment)

 - Create the application Deployment
```
kubectl apply -f ./app-v1-base/app.yaml
```

 - Deploy a ClusterIP service to expose the application (internally)
```
kubectl apply -f ./app-v1-base/svc.yaml
```

---

## 2) Allow access to the application "v1" trough the ingress controller

#### Create alias on Route53 to access the application

 - Use an existent hosted zone (seladevops.com in my case) to create a record set with the following details:
```
Name: grpc.green.seladevops.com
Type: A - IPv4 address
Alias: Yes
Alias Target: <ingress load balancer dns>
Routing policy: simple
Evaluate target health: no
```

 - Test the configuration by browsing to the created alias (you should receive error 404 from nginx):
```
https://grpc.green.seladevops.com
```

#### Create a SSL certificate for TLS termination

 - Create a self signed certificate by run (grpc.seladevops.com in my case):
```
mkdir -p ./_certificates/green
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./_certificates/green/tls.key -out ./_certificates/green/tls.crt -subj "/CN=grpc.green.seladevops.com/O=grpc.green.seladevops.com"
```

 - Create a kubernetes secret to store the certificate:
```
kubectl create secret tls green-cert --key ./_certificates/green/tls.key --cert ./_certificates/green/tls.crt
```
 
#### Expose the application trough the ingress controller

 - Deploy a ingress resources to allow reach the application
```
kubectl apply -f ./ingress/ingress-green.yaml
```

#### Browse to the application

 - Check all services provided by the gRPC server by run:
```
grpcurl -insecure grpc.green.seladevops.com:443 list
```

 - Check the interface for calling the helloworld.Greeter service:
```
grpcurl -insecure grpc.green.seladevops.com:443 list helloworld.Greeter
```

 - Query the description of specific protocol parameters of the SayHello interface:
```
grpcurl -insecure grpc.green.seladevops.com:443 describe helloworld.Greeter.SayHello
```

 - Call the SayHello interface and specify the name parameter
```
grpcurl -insecure -d '{"name": "gRPC"}' grpc.green.seladevops.com:443 helloworld.Greeter.SayHello
```
```
grpcurl -insecure -d '{"name": "World!!"}' grpc.green.seladevops.com:443 helloworld.Greeter.SayHello
```

---

## 3) Configure TrafficSplit to manage the routing to the application

 - Let's create a new ClusterIP service to reach the current version "v1"
```
kubectl apply -f ./app-v1-green/svc-v1.yaml
```

 - Now create the TrafficSplit resource to send all the traffic received by the base service to the "v1" service
```
kubectl apply -f ./trafficsplit/trafficsplit-green-v1.yaml
```

 - Finally let's update the application deployment to add the label "version: v1"
```
kubectl apply -f ./app-v1-green/app-v1.yaml
```

---

## 4) Deploy a new version of the application "v2" (blue version)

 - Create the application Deployment
```
kubectl apply -f ./app-v2-blue/app-v2.yaml
```

 - Deploy a ClusterIP service to expose the application (internally)
```
kubectl apply -f ./app-v2-blue/svc-v2.yaml
```

---

## 5) Allow access to the application "v2" trough the ingress controller

#### Create alias on Route53 to access the application

 - Use an existent hosted zone (seladevops.com in my case) to create a record set with the following details:
```
Name: grpc.blue.seladevops.com
Type: A - IPv4 address
Alias: Yes
Alias Target: <ingress load balancer dns>
Routing policy: simple
Evaluate target health: no
```

 - Test the configuration by browsing to the created alias (you should receive error 404 from nginx):
```
https://grpc.blue.seladevops.com
```

#### Create a SSL certificate for TLS termination

 - Create a self signed certificate by run (grpc.seladevops.com in my case):
```
mkdir ./_certificates/blue
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./_certificates/blue/tls.key -out ./_certificates/blue/tls.crt -subj "/CN=grpc.blue.seladevops.com/O=grpc.blue.seladevops.com"
```

 - Create a kubernetes secret to store the certificate:
```
kubectl create secret tls blue-cert --key ./_certificates/blue/tls.key --cert ./_certificates/blue/tls.crt
```
 
#### Expose the application trough the ingress controller using TrafficSplit

 - Create a main service for TrafficSplit resource:
```
kubectl apply -f ./trafficsplit/svc-blue.yaml
```

 - Create the TrafficSplit resource to redirect traffic to version "v2":
```
kubectl apply -f ./trafficsplit/trafficsplit-blue-v2.yaml
```

 - Deploy a ingress resource to expose the application
```
kubectl apply -f ./ingress/ingress-blue.yaml
```

#### Browse to the application

 - Check all services provided by the gRPC server by run:
```
grpcurl -insecure grpc.blue.seladevops.com:443 list
```

 - Check the interface for calling the helloworld.Greeter service:
```
grpcurl -insecure grpc.blue.seladevops.com:443 list helloworld.Greeter
```

 - Query the description of specific protocol parameters of the SayHello interface:
```
grpcurl -insecure grpc.blue.seladevops.com:443 describe helloworld.Greeter.SayHello
```

 - Call the SayHello interface and specify the name parameter
```
grpcurl -insecure -d '{"name": "gRPC"}' grpc.blue.seladevops.com:443 helloworld.Greeter.SayHello
```
```
grpcurl -insecure -d '{"name": "World!!"}' grpc.blue.seladevops.com:443 helloworld.Greeter.SayHello
```

---

## 6) Switch traffic to the new version

 - Let's update the TrafficSplit resource to update the green version to route the traffic to "v2"
```
kubectl apply -f ./trafficsplit/trafficsplit-green-v2.yaml
```

 - Browse to the green application to check the deployed version:
```
grpcurl -insecure -d '{"name": "gRPC"}' grpc.green.seladevops.com:443 helloworld.Greeter.SayHello
```

---

 7) Rollback the deployment

 - Let's update the TrafficSplit resource to update the green version to route the traffic to "v1"
```
kubectl apply -f ./trafficsplit/trafficsplit-green-v1.yaml
```

 - Browse to the green application to check the deployed version:
```
grpcurl -insecure -d '{"name": "gRPC"}' grpc.green.seladevops.com:443 helloworld.Greeter.SayHello
```

---
