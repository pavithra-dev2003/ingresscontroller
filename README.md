# ingresscontroller
An Ingress Controller is a Kubernetes resource that watches the Ingress API objects and routes external traffic to the appropriate backend services. It acts as a reverse proxy and load balancer.
We are using NGINX Ingress Controller in this project.
Installation (Using Helm)
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace
