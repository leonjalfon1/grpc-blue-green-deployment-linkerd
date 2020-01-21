# Blue/Green Deployment for gRPC Application with Linkerd
---


 - In this demo we will see how to implement blue/green deployment for a gRPC application.
 - Linkerd (service mesh) is installed in the cluster and is used to manage the microservices.
 - In general what we will do is the following:
 
    1) Deploy the application "v1" (before blue/green deployment)
    2) Allow access to the application trough the ingress controller
    3) Configure TrafficSplit to manage the routing to the application (green version)
    4) Deploy a new version of the application "v2" (blue version)
    5) Allow access to the application trough the ingress controller
    6) Switch traffic to the new version
    7) Rollback the deployment

 - The demo instructions are based on AWS EKS but apply for every kubernetes cluster
 - For instructions about configure the environment see the "prerequisites" folder
