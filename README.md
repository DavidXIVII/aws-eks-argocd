# Amazon EKS Cluster
This repository contains source code to provision an EKS cluster in AWS using Terraform. 

## Inspiration:
https://github.com/LukeMwila/amazon-eks-cluster

## Prerequisites
* AWS account
* AWS profile configured with CLI on local machine
* [Terraform](https://www.terraform.io/)
* [kubectl](https://kubernetes.io/docs/tasks/tools/)

## Project Structure

```
├── README.md
├── eks
|  ├── cluster.tf
|  ├── cluster_role.tf
|  ├── cluster_sg.tf
|  ├── node_group.tf
|  ├── node_group_role.tf
|  ├── node_sg.tf
|  └── vars.tf
├── main.tf
├── provider.tf
├── raw-manifests
|  ├── aws-auth.yaml
|  ├── pod.yaml
|  └── service.yaml
├── variables.tf
└── vpc
   ├── control_plane_sg.tf
   ├── data_plane_sg.tf
   ├── nat_gw.tf
   ├── output.tf
   ├── public_sg.tf
   ├── vars.tf
   └── vpc.tf
```

## Provision Infrastructure
Review the *main.tf* to update the node size configurations (i.e. desired, maximum, and minimum). When you're ready, run the following commands:
1. `terraform init` - Initialize the project, setup the state persistence (whether local or remote) and download the API plugins.
2. `terraform plan` - Print the plan of the desired state without changing the state.
3. `terraform apply` - Print the desired state of infrastructure changes with the option to execute the plan and provision. 

## Connect To Cluster
Using the same AWS account profile that provisioned the infrastructure, you can connect to your cluster by updating your local kube config with the following command:
`aws eks --region <aws-region> update-kubeconfig --name <cluster-name>`

## Launch ArgoCD
1. 
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## Install ArgoCD CLI
[CLI Installation](https://argo-cd.readthedocs.io/en/stable/cli_installation)

## Access ArgoCD API Server

1. Has LoadBalancer
`kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'`

## Login to ArgoCD using CLI

1. First we need to get our encoded password
`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo`

2. Login via the CLI
`argocd login <aws lb url*>`
* The AWS loadbalancer url can be found under the LB section in EC2 control plane or by running 
`kubectl get svc --all-namespaces`

3. Login in with username admin and password granted at the beginning.

4. Update the password
`argocd account update-password`

## Deploy a manifest using the cli

1. 
```argocd app create weatherapp --repo https://github.com/DavidXIVII/weather-deployment-k8s.git --path eks --dest-server https://kubernetes.default.svc --dest-namespace default
```

