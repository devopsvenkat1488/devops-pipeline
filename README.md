# devops-pipeline
This repo is to build a python application using terraform and Kubernetes

DevOps Pipeline: Terraform, Kubernetes, and Python

This repository demonstrates an end-to-end DevOps pipeline using Terraform, Kubernetes, and Python. The pipeline provisions resources, deploys a Python application to a Kubernetes cluster, and exposes it using a NodePort service.

Folder Structure
.
├── terraform/
│   ├── main.tf
│   ├── variables.tf
├── app/
│   ├── Dockerfile
│   └── app.py
├── kubernetes/
│   ├── deployment.yaml
│   ├── service.yaml
├── python-scripts/
│   └── deploy_app.py


Pre-requisites

A Kubernetes cluster running locally (e.g., Minikube, kind, or MicroK8s).

Install the following tools:

1. Terraform

2. kubectl

3. Python 3.x with pip and virtualenv

4. Docker

Steps to Set Up the Pipeline

1. Create a Simple Python Web Application

 1. Write the Python App: Create app/app.py:
 
from flask import Flask
app = Flask(__name__)

@app.route('/')
def home():
    return "Hello, World! Welcome to the DevOps Pipeline Demo."

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

2. Dockerize the Application: Create app/Dockerfile:

FROM python:3.9-slim
WORKDIR /app
COPY app.py /app
RUN pip install flask
CMD ["python", "app.py"]

3. Build and Test the Docker Image:

cd app
docker build -t python-app .
docker run -p 5000:5000 python-app



2. Provision Kubernetes Resources with Terraform


1. Terraform Configuration: Create terraform/main.tf:

provider "kubernetes" {
  config_path = "~/.kube/config"
}

resource "kubernetes_namespace" "dev" {
  metadata {
    name = "dev"
  }
}

2. Initialize and Apply Terraform:

cd terraform
terraform init
terraform apply

3. Deploy the Application to Kubernetes

1. Write Kubernetes Deployment: Create kubernetes/deployment.yaml:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-app
  namespace: dev
spec:
  replicas: 2
  selector:
    matchLabels:
      app: python-app
  template:
    metadata:
      labels:
        app: python-app
    spec:
      containers:
      - name: python-container
        image: python-app:latest
        ports:
        - containerPort: 5000

2. Write Kubernetes NodePort Service: Create kubernetes/service.yaml:

apiVersion: v1
kind: Service
metadata:
  name: python-app-service
  namespace: dev
spec:
  selector:
    app: python-app
  ports:
  - protocol: TCP
    port: 5000
    targetPort: 5000
    nodePort: 30007
  type: NodePort

3. Apply Kubernetes Manifests:

kubectl apply -f kubernetes/



4. Automate with Python

1. Install Python Dependencies:

python -m venv venv
source venv/bin/activate
pip install kubernetes


2. Python Automation Script: Create python-scripts/deploy_app.py:

from kubernetes import client, config, utils

def deploy():
    config.load_kube_config()
    k8s_client = client.ApiClient()

    # Apply manifests
    utils.create_from_yaml(k8s_client, "kubernetes/deployment.yaml")
    utils.create_from_yaml(k8s_client, "kubernetes/service.yaml")

    print("Deployment and Service created successfully!")

if __name__ == "__main__":
    deploy()

3. Run the Script:

python python-scripts/deploy_app.py


5. Access the Application

1. Verify Kubernetes Resources:

kubectl get pods -n dev
kubectl get svc -n dev

2. Access the Application:

Open a browser and navigate to:

http://<Node_IP>:30007



Notes

Ensure Docker is running and the image python-app is available on the Kubernetes node(s).

To rebuild the Docker image and push it to a registry, update the deployment manifest with the image's registry path.

License

This project is licensed under the MIT License.
