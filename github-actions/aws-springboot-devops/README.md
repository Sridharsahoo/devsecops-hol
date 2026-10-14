# AWS DevOps Demo

Simple NGINX application used for demonstrating an end-to-end Jenkins CI/CD pipeline.

## Application

Version: 1.0

Environment: Development

Deployment: Jenkins + Docker + Kubernetes

## Expose port-forward on all network interfaces for testing and access from browser
## kubectl port-forward svc/aws-devops-demo 8080:80 --address 0.0.0.0

## Access it from browser directly without port forwarding after nginx configuration

Browser
http://EC2-PUBLIC-IP:8080 

EC2 host — Nginx reverse proxy
Listens on TCP 8080

Minikube NodePort
192.168.49.2:30080

Kubernetes Service → application pods

Step 1: Install Nginx on the Minikube EC2 host
Run these commands on ip-172-31-10-111:

sudo apt-get update
sudo apt-get install -y nginx

Step 2: Configure Nginx
Create a reverse proxy configuration:

sudo tee /etc/nginx/sites-available/aws-devops-demo > /dev/null <<'EOF'
server {
    listen 8080;
    server_name _;

    location / {
        proxy_pass http://192.168.49.2:30080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
EOF

Enable it and validate the configuration:

sudo ln -s /etc/nginx/sites-available/aws-devops-demo \
  /etc/nginx/sites-enabled/aws-devops-demo

sudo nginx -t
sudo systemctl reload nginx

Step 3: Test from the EC2 host

curl -v http://127.0.0.1:8080/

Then open this URL in your browser

http://YOUR-EC2-PUBLIC-IP:8080