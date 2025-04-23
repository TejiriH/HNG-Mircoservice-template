# HNG12 DevOps Stage4 Task

This Repo Contains the code for a microservice application comprising of several components communicating to each other. In other words, this is an example of microservice. These microservices are written in different languages.

The app itself is a simple TODO app that additionally authenticates users.

## Components

1. [Frontend](/frontend) part is a Javascript application, provides UI. Created with [VueJS](http://vuejs.org)
2. [Auth API](/auth-api) is written in Go and provides authorization functionality. Generates JWT tokens to be used with other APIs.
3. [TODOs API](/todos-api) is written with NodeJS, provides CRUD functionality ove user's todo records. Also, it logs "create" and "delete" operations to Redis queue, so they can be later processed by [Log Message Processor](/log-message-processor).
4. [Users API](/users-api) is a Spring Boot project written in Java. Provides user profiles. Does not provide full CRUD for simplicity, just getting a single user and all users.
5. [Log Message Processor](/log-message-processor) is a very short queue processor written in Python. It's sole purpose is to read messages from Redis queue and print them to stdout


The diagram describes the various components and their interactions.
![microservice-app-example](https://user-images.githubusercontent.com/1905821/34918427-a931d84e-f952-11e7-85a0-ace34a2e8edb.png)

Note: 3 different login details are provided in the .env file 

## License

MIT


# 🧩Microservices Deployment on AWS EKS with ArgoCD, Prometheus & Grafana Monitoring
## Overview
This project demonstrates how to deploy a microservices-based application on an AWS EKS cluster using ArgoCD for automated deployments. It includes setting up Prometheus and Grafana for monitoring, and configuring Node Exporter to scrape system metrics.

## 🌐 Project Structure
Microservices involved:

todos-image (Node.js)

auth-image (Go)

log-image (Python, message processor)

frontend (Vue.js)

redis (In-memory store)

## Prerequisites
AWS account with IAM permissions to create and manage EKS resources

eksctl, kubectl, argocd, and aws CLI tools installed

A local machine for interacting with the cluster

Docker installed for building and pushing images to AWS ECR

## Table of Contents
Setting Up the EKS Cluster

Setting Up ArgoCD

Deploying the Microservices

Configuring Ingress for Frontend

Installing Prometheus, Grafana, and Node Exporter

Configuring Prometheus to Scrape Node Exporter

Conclusion


🚀 Cluster Setup with eksctl
We created an EKS cluster named microservice using the following command:

```
eksctl create cluster \
  --name microservice \
  --region eu-north-1 \
  --nodegroup-name microservice-nodes \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --managed
  ```
This command creates an EKS cluster with three nodes in the eu-north-1 region. You can adjust the number of nodes as needed.

Configure kubectl:

Once the cluster is created, configure kubectl to use the new cluster:

```
aws eks --region eu-north-1 update-kubeconfig --name microservice
```

## 🛠️ ArgoCD Installation and Setup

Install ArgoCD in argocd namespace:

> kubectl create namespace argocd

> kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

Expose ArgoCD with a LoadBalancer:

> kubectl patch svc argocd-server -n argocd --patch "$(cat patch-argocd-server.yaml)"

The patch-argocd-server.yaml file in the k8s/argocd directory

Get the external IP:

> kubectl get svc -n argocd

Login to ArgoCD
Get the admin password:

> kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d

Access the UI via the LoadBalancer IP, and login with username admin and the above password.


## 📥 ArgoCD Application Deployment
Each microservice's manifest was stored in a GitHub repo. We created ArgoCD apps pointing to those manifests.

Example argocd.yaml:

> kubectl apply -f argocd.yaml

## ⚙️ Ingress Controller Setup
Deployed NGINX ingress controller to expose the frontend:


> kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/aws/deploy.yaml

Created ingress resources for the frontend: k8s/ingress/ingress.yaml

> kubectl apply -f ingress.yaml

## 📈 Monitoring with Prometheus and Grafana

Download Prometheus from Prometheus Downloads, and extract it:
```
wget https://github.com/prometheus/prometheus/releases/download/v2.53.4/prometheus-2.53.4.linux-amd64.tar.gz
tar -xvzf prometheus-2.53.4.linux-amd64.tar.gz
cd prometheus-2.53.4.linux-amd64
./prometheus --config.file=prometheus.yml
```
```
wget https://dl.grafana.com/enterprise/release/grafana-enterprise_11.6.0_amd64.deb
sudo dpkg -i grafana-enterprise_11.6.0_amd64.deb
sudo systemctl daemon-reload
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```
### Install Node Exporter:

Download Node Exporter from Node Exporter Downloads, and extract it:

```
wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz
tar -xvzf node_exporter-1.9.1.linux-amd64.tar.gz
cd node_exporter-1.9.1.linux-amd64
./node_exporter
```
Configuring Prometheus to Scrape Node Exporter
Modify prometheus.yml:

Add the Node Exporter job to the prometheus.yml file under scrape_configs:

```
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['server-ip:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: 
         - '<node-ip-1>:<nodeport>'
         - '<node-ip-2>:<nodeport>'
```
Restart Prometheus:

Restart Prometheus to apply the changes:


> ./prometheus --config.file=prometheus.yml
Access Prometheus:

Access Prometheus by visiting http://server-ip:9090 in your browser.

Check Node Exporter targets in:
Status > Targets

### Access Grafana:
> http://your-server-ip:3000

Default login:

Username: admin

Password: admin (you'll be prompted to change it)

### ➕ Add Prometheus as a Data Source
Log in to Grafana.

Go to Settings > Data Sources.

Choose Prometheus.

URL: http://server-ip:9090

Save and test.

### Import Node Exporter Dashboard in Grafana
Go to + > Import Dashboard.

Enter 1860 (Node Exporter Full dashboard ID).

Select the Prometheus data source.

Click Import.



Conclusion
This setup includes the full deployment of a microservices-based application using AWS EKS, ArgoCD for continuous deployment, and Prometheus and Grafana for monitoring. The Node Exporter was installed manually, and Prometheus was configured to scrape system metrics for monitoring.



