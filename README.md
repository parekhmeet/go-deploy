# go-deploy

This repository hosts the code for a sample Go web application named **go-deploy**. This document will guide you through the steps to build, run, and deploy the application, as well as provide an overview of the CI/CD pipeline.

## Prerequisites

Before you begin, ensure you have the following installed on your system:
- [Go](https://golang.org/dl/)
- [Docker](https://www.docker.com/get-started)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Argo CD CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/)
- [AWS CLI](https://aws.amazon.com/cli/)
- [eksctl](https://eksctl.io/installation/)

## Installation

Clone the repository:

```
git clone https://github.com/yourusername/go-deploy.git
cd go-deploy
```

Building the Application:

- To build the Go application, run the following command:
- This will compile the code and generate an executable named go-deploy.
```
go build -o go-deploy
```

Running the Application: 

- To run the application, execute the following command:
- By default, the application will start on port 8080. You can access it by navigating to http://localhost:8080 in your web browser.
```
./go-deploy
```

## Continuous Integration (CI) : 
- Continuous Integration is managed using GitHub Actions. The CI pipeline is configured to run tests and build the application whenever changes are pushed to the repository.
- The GitHub Actions workflow file is located at .github/workflows/ci.yml.

## Continuous Deployment (CD) : 
- Continuous Deployment is handled using Argo CD. The deployment process is triggered once the CI pipeline passes.
- The Argo CD application configuration is located in the deploy/ directory.

## Deploying to AWS EKS : 
- To deploy the Go web application to an AWS EKS cluster, follow these steps:

Configure AWS CLI:
```
aws configure
```

Create an EKS cluster: 
```
eksctl create cluster --name $NAME_OF_CLUSTER --region $REGION
```

Deploy deployment / service / ingress : 
```
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```
Deploy NGINX AWS Ingress Controller : 
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/aws/deploy.yaml
```
Update /etc/hosts with the address given by the LB from the NGINX INGRESS CONTROLLER : 
```
kubectl get ing
vim /etc/hosts #update the Address and map it with the hostname for example : #####.elb.ap-southeast-1.amazonaws.com go-deploy.local
```

## Creating Helm Charts

To package and deploy the Go web application using Helm, follow these steps:

1. **Install Helm:**

   If you haven't installed Helm yet, follow the instructions [here](https://helm.sh/docs/intro/install/).

2. **Create a Helm Chart:**

   Navigate to the root directory of your project and create a new Helm chart:

   ```
   helm create go-deploy
   ```

## Configure Github Actions and Argo CD for CI/CD : 

Create .github/workflows/ci.yml in the root directory. 
Configure CI : 
- build
- test
- Upload the image to docker hub.
- update newtag in helm-chart
- (Configure docker and github token in GitHub Repository settings -> secrets -> actions -> New repository secret)

Configure CD : 

Install Argo CD on the EKS cluster:
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```
- Access Argo CD UI :
```
kubectl get svc -n argocd
```
- Fetch the ArgoCD credentials :
  ```
  kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
  ```
- Login to ArgoCD and configure new app

## Access and update the go web application :

- Access the web app using the address updated in the /etc/hosts file.
- Now the CI/CD will update the application on Git push event.

## Credits  
- Thanks to Abhishek Veeramala for the amazing tutorial.
