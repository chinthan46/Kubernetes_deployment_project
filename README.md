🚀 Kubernetes Deployment of Dockerized Application

Deploying a Dockerized Python/Node application on Kubernetes (Minikube) with Deployments, Services, ConfigMaps, and Horizontal Pod Autoscaler (HPA).

📘 Project Overview

This project demonstrates a complete DevOps workflow by deploying a containerized application on a local Kubernetes cluster. It includes:

Containerization using Docker

Deployment on Minikube

Kubernetes Deployment, Service, ConfigMap

Horizontal Pod Autoscaling based on CPU

Load testing to auto-scale pods

Accessing the app via NodePort / Minikube service


🏗️ Project Architecture

                
                 +-----------------------------+
                 |         ConfigMap           |
                 |  (Environment Variables)    |
                 +--------------+--------------+
                                |
                                v
          +--------------------+     +----+-----+
         |  User / Browser    | --> | Service  |  (NodePort - exposes app)
          +--------------------+     +----+-----+
                                  |
                                  
                                  v
                                  
                         +--------+--------+
                         |   Deployment     |
                         | (Multiple Pods)  |
                         +--------+---------+
                                  |
                                  v
                                  
                          +-------+------+
                          |  Dockerized   |
                          |   Application |
                          +--------------+
                          


HPA monitors CPU usage of the Deployment and scales pods up/down automatically.

📂 Project Structure
k8s/
  configmap.yaml
  deployment.yaml
  service.yaml
  hpa.yaml
Dockerfile
app.py


🚀 Step 1: Start Minikube
minikube start
kubectl get nodes

🐳 Step 2: Build Docker Image inside Minikube
eval $(minikube docker-env)
docker build -t myapp:latest .

⚙️ Step 3: Apply Kubernetes Manifests
1. ConfigMap

k8s/configmap.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  APP_MESSAGE: "Hello from Kubernetes!"


Apply:

kubectl apply -f k8s/configmap.yaml

2. Deployment

k8s/deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myapp:latest
          ports:
            - containerPort: 5000
          envFrom:
            - configMapRef:
                name: app-config
          resources:
            requests:
              cpu: "100m"
            limits:
              cpu: "300m"


Apply:

kubectl apply -f k8s/deployment.yaml

3. Service

k8s/service.yaml

apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
    - port: 5000
      targetPort: 5000
      nodePort: 30007


Apply:

kubectl apply -f k8s/service.yaml


Access App:

minikube service myapp-service

📈 Step 4: Horizontal Pod Autoscaler (HPA)

Enable Metrics Server:

minikube addons enable metrics-server


k8s/hpa.yaml

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp-deployment
  minReplicas: 2
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50


Apply:

kubectl apply -f k8s/hpa.yaml
kubectl get hpa

🔥 Step 5: Load Testing to Trigger Scaling

Start a load generator pod:

kubectl run -it --rm load-generator --image=busybox -- sh


Inside:

while true; do wget -q -O- http://myapp-service:5000; done


Watch HPA respond:

kubectl get hpa
kubectl get pods -w


Pods will scale from 2 → 3 → 4 → 5 based on CPU usage.
