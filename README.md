
# Kubernetes Custom Web App with Velero Backup and Restore

This project demonstrates how to deploy a custom web application on a Kubernetes cluster using **Nginx**, manage DNS using **DuckDNS**, and integrate **Velero** for backup and restore operations. The application runs on a **Google Kubernetes Engine (GKE)** cluster with a static IP and a custom domain provided by DuckDNS.

## Overview

In this project, we:
1. Create a custom web application deployed using **Nginx** on a GKE cluster.
2. Map a **DuckDNS** subdomain to the GKE service using a reserved static IP.
3. Set up **Velero** to back up the web application's namespace and restore it after deletion.

## Prerequisites

- Google Cloud account with access to **Google Kubernetes Engine (GKE)**
- **gcloud** and **kubectl** CLI installed
- **Docker** for building custom images
- A **DuckDNS** account (free dynamic DNS provider)
- **Velero** CLI installed

## Setup Steps

### 1. Create a GKE Cluster

First, create a GKE cluster:

```bash
gcloud container clusters create my-cluster \
    --num-nodes=3 \
    --zone us-east1-b
```

Connect your local environment to the GKE cluster:

```bash
gcloud container clusters get-credentials my-cluster --zone us-east1-b
```

### 2. Deploy a Custom Nginx Web App

1. **Create a Custom Web Page:**

   Save the following content in a file named `index.html`:

   ```html
   <!DOCTYPE html>
   <html>
   <head>
     <title>My Kubernetes Web App</title>
     <style>
       body { font-family: Arial, sans-serif; text-align: center; margin-top: 50px; }
       h1 { color: #333; }
       p { color: #666; }
     </style>
   </head>
   <body>
     <h1>Welcome to My Custom Web App on Kubernetes!</h1>
     <p>This page is running on a Kubernetes cluster using Nginx.</p>
     <p>Check out my projects on <a href="https://github.com/your-github-username">GitHub</a>.</p>
   </body>
   </html>
   ```

2. **Create a Docker Image:**

   Create a `Dockerfile` for Nginx:

   ```Dockerfile
   FROM nginx:alpine
   COPY index.html /usr/share/nginx/html/index.html
   ```

   Build and push the Docker image to Docker Hub:

   ```bash
   docker build -t <your-dockerhub-username>/custom-nginx .
   docker push <your-dockerhub-username>/custom-nginx
   ```

3. **Deploy the Nginx Application in GKE:**

   Create a Kubernetes deployment using your custom image:

   ```bash
   kubectl create namespace my-web-app
   kubectl create deployment nginx --image=<your-dockerhub-username>/custom-nginx --namespace=my-web-app
   kubectl expose deployment nginx --port=80 --target-port=80 --type=LoadBalancer --namespace=my-web-app
   ```

4. **Reserve a Static IP in GCP:**

   ```bash
   gcloud compute addresses create nginx-static-ip --region us-east1
   ```

   Update the Nginx service to use the static IP:

   ```bash
   kubectl patch svc nginx -n my-web-app -p '{"spec": {"loadBalancerIP": "35.196.170.189"}}'
   ```

### 3. Set Up DuckDNS for a Custom Domain

1. Sign up for a free account at [DuckDNS](https://www.duckdns.org/).
2. Create a subdomain (e.g., `kushalesh.duckdns.org`).
3. Map the subdomain to your static IP (`35.196.170.189`).
4. Verify DNS propagation:

   ```bash
   nslookup kushalesh.duckdns.org
   ```

Now, you can access your web app at `http://kushalesh.duckdns.org`.

### 4. Install Velero for Backup and Restore

1. **Install Velero in your GKE cluster:**

   ```bash
   velero install \
      --provider gcp \
      --bucket <your-gcs-bucket-name> \
      --secret-file ./key.json \
      --backup-location-config serviceAccount=./key.json \
      --use-volume-snapshots=false \
      --plugins velero/velero-plugin-for-gcp:v1.5.0 \
      --wait
   ```

2. **Verify Velero Installation:**

   ```bash
   kubectl get pods -n velero
   ```

### 5. Backup and Restore the Web App

1. **Create a Backup of the Namespace:**

   ```bash
   velero backup create my-web-app-backup --include-namespaces my-web-app
   ```

2. **Simulate a Disaster:**

   Delete the namespace:

   ```bash
   kubectl delete namespace my-web-app
   ```

3. **Restore the Backup:**

   ```bash
   velero restore create --from-backup my-web-app-backup
   ```

4. **Verify the Restoration:**

   ```bash
   kubectl get all --namespace=my-web-app
   ```

Your web application should be restored and accessible again via `http://kushalesh.duckdns.org`.

## Conclusion

This project showcases how to:
- Deploy a custom web app using Nginx on GKE.
- Use DuckDNS to manage DNS for a Kubernetes service with a static IP.
- Implement Velero for Kubernetes backup and restore functionality.


