# Blue/Green Deployment for GRPC App with Linkerd
---

## Prerequisites

 1) Install required client tools
 2) Provision an EKS cluster
 3) Install linkerd
 4) Deploy the nginx ingress controller
 
---

## 1) Install Client Tools

 - kubectl
```
sudo curl --silent --location -o /usr/local/bin/kubectl https://amazon-eks.s3-us-west-2.amazonaws.com/1.14.6/2019-08-22/bin/linux/amd64/kubectl
sudo chmod +x /usr/local/bin/kubectl
```
 
 - aws cli
```
url "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" -o "awscli-bundle.zip"
unzip awscli-bundle.zip
sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
aws configure
```
 
 - iam-authenticator
```
curl -o aws-iam-authenticator https://amazon-eks.s3-us-west-2.amazonaws.com/1.14.6/2019-08-22/bin/linux/amd64/aws-iam-authenticator
chmod +x ./aws-iam-authenticator
mkdir -p $HOME/bin && cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=$PATH:$HOME/bin
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
```

 - eksctl
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/latest_release/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv -v /tmp/eksctl /usr/local/bin
```

 - grpcurl
```
go get github.com/fullstorydev/grpcurl
go install github.com/fullstorydev/grpcurl/cmd/grpcurl
export PATH=$PATH:$HOME/go/bin
```

---

## 2) Cluster Provisioning

 -  To create an empty cluster, run:
```
eksctl create cluster -f eks-cluster.yaml
```

---

## 3) Install Linkerd

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

## 4) Install nginx ingress controller

 - Install the nginx ingress controller:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.27.1/deploy/static/mandatory.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.27.1/deploy/static/provider/aws/service-l4.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.27.1/deploy/static/provider/aws/patch-configmap-l4.yaml
```

 - Retrieve the ingress controller load balancer DNS:
```
kubectl get svc -n ingress-nginx
```

 - Browse to the load balancer DNS and ensure that you get the 404 error from nginx (the load balancer can take a while to be available)

---
