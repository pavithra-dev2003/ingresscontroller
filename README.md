# Ingress Setup for User Management and Nginx Apps
This guide explains how to deploy the NGINX Ingress Controller with a LoadBalancer service and configure Ingress rules to expose three applications:

* User Management WebApp (with MySQL backend) → /

* App1 Nginx → /app1

* App2 Nginx → /app2

## Prerequisites
* A Kubernetes cluster (AKS, EKS, GKE, or Minikube).
* kubectl configured to connect to your cluster.
* Helm installed.

### Install NGINX Ingress Controller
Add the ingress-nginx Helm repo:
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

### Deploy the Ingress controller with a LoadBalancer service type:
```
helm install nginx-ingress ingress-nginx/ingress-nginx -f values.yaml
```
### Customizing the Chart Before Installing. 
```
helm show values ingress-nginx/ingress-nginx
```

### Here, values.yaml contains the configuration:
```
controller:
  ingressClass: nginx
  service:
    type: LoadBalancer
  containerPort:
    http: 80
    https: 443
rbac:
  create: true
```

### values.yaml configuration

This file tells Helm how to install the NGINX Ingress Controller.
ingressClass: nginx → defines the ingress class name (must match your Ingress manifest).
service.type: LoadBalancer → exposes the Ingress Controller to the internet with an external IP.
containerPort.http/https → sets HTTP (80) and HTTPS (443) ports.
rbac.create: true → creates role-based access permissions automatically.

#### Applications and Services

You need to deploy three apps, and each must have a ClusterIP Service so the Ingress can forward traffic internally:
App1 (Nginx) → app1-nginx-clusterip-service
App2 (Nginx) → app2-nginx-clusterip-service
User Management WebApp (with MySQL + PVC + ConfigMap + Secret) → usermgmt-webapp-clusterip-service

### On Minikube, use:

minikube addons enable ingress

### Deploy Applications and Services

* App1 Deployment + Service 
* App2 Deployment + Service 
* User Management WebApp + MySQL + ConfigMap + PVC + Secret

### Make sure each app has a ClusterIP Service:

* usermgmt-webapp-clusterip-service

* app1-nginx-clusterip-service

* app2-nginx-clusterip-service

#### Apply Ingress Resource

Create an Ingress resource that maps different paths to different services:
#### Ingress API Version and Kind
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-service
```

apiVersion: Uses networking.k8s.io/v1 which is the stable API version for Ingress.
kind: Ingress specifies that this manifest defines an Ingress resource.
metadata.name: Names the Ingress resource as ingress-service.
This tells Kubernetes we are creating an Ingress resource named ingress-service.

#### Ingress Class
```
spec:
  ingressClassName: nginx

```
spec.ingressClassName: Defines which Ingress Controller should handle this Ingress resource.
Here it is set to nginx, meaning the NGINX Ingress Controller will process and route incoming requests.
Other options (if installed) could include azure/application-gateway, haproxy, etc.

#### Rules Section (Path-Based Routing)
```
  rules:
  - http:
      paths:
```

rules: Defines how external requests are routed to backend services.
http: Specifies that these are HTTP routing rules.
paths: Contains different path rules that will map specific URL paths to backend services inside the cluster.

#### Default Root Path (/ → usermgmt-webapp)
```
      - path: /
        pathType: Prefix
        backend:
          service:
            name: usermgmt-webapp-clusterip-service
            port:
              number: 80
```

path: / → Requests with root URL (e.g., http://<EXTERNAL-IP>/) are handled here.
pathType: Prefix → Means any request starting with / will be routed.
backend.service.name: usermgmt-webapp-clusterip-service → The traffic will be forwarded to this Service.
port.number: 80 → Uses service port 80 of the user management webapp.
Example: Visiting http://<LoadBalancer-IP>/ will show the User Management Web Application.

### App1 Path (/app1 → app1 service)
```
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-nginx-clusterip-service
            port:
              number: 80
```

path: /app1 → Any request starting with /app1 will match.
backend.service.name: app1-nginx-clusterip-service → Routes to App1 Service.
port.number: 80 → Exposes App1 on port 80 internally.
Example: Visiting http://<LoadBalancer-IP>/app1 will show the App1 application.

#### App2 Path (/app2 → app2 service)
```
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-nginx-clusterip-service
            port:
              number: 80
```

path: /app2 → Any request starting with /app2 will match.
backend.service.name: app2-nginx-clusterip-service → Routes to App2 Service.
port.number: 80 → Exposes App2 on port 80 internally.
Example: Visiting http://<LoadBalancer-IP>/app2 will show the App2 application.


#### Apply it:
```
kubectl apply -f ingress.yaml
```
#### Verify Deployment

Check that the Ingress Controller is running:
```
kubectl get pods -n ingress-nginx
```

#### Check the LoadBalancer external IP:
```
kubectl get svc -n ingress-nginx
```

#### Check your Ingress rules:
```
kubectl get ingress
```
#### Access Applications

Once the LoadBalancer IP is assigned, access the apps:

* User Management WebApp → http://<EXTERNAL-IP>/

* App1 → http://<EXTERNAL-IP>/app1

* App2 → http://<EXTERNAL-IP>/app2

✅ Summary

NGINX Ingress Controller handles incoming traffic.

LoadBalancer service exposes the Ingress Controller externally.

Ingress rules route traffic to different applications based on URL path.
