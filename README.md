
# Kubernetes Documentation for Frappe/ERPNext with Custom App Deployment

This guide walks you through deploying Frappe/ERPNext with Custom App in a Kubernetes environment using custom Docker images, NFS for storage, and Helm for installation and upgrades.

## Prerequisites

Before starting, ensure you have the following tools installed:
- **Minikube**: To run a local Kubernetes cluster.
- **Docker**: To build and push Docker images.
- **Kubectl**: To interact with Kubernetes.
- **Helm**: To manage Kubernetes applications.

## 1. Build the Docker Image

To deploy Frappe/ERPNext, first, you need to build the Docker image.

1. **Pull the pre-built custom Docker image:**

   Pull the custom image from the Docker registry:

   ```bash
   docker pull kraegavila/frappe-custom:v.0.6
   ```

2. **Add Custom Apps:**

   To add a custom app, you need to rebuild the Docker image with the app included. Modify your `apps.json` to include the custom app, for example:

   ```json
   [
     {
       "url": "https://github.com/frappe/erpnext",
       "branch": "version-15"
     },
     {
       "url": "https://github.com/ellavondegurechaff/asnsmri",
       "branch": "develop"
     }
   ]
   ```

   Then, rebuild the Docker image with:

   ```bash
   docker build -t kraegavila/frappe-custom:v.0.6 .
   ```

## 2. Compose the Docker Image

Once the Docker image is built, you can use Docker Compose to start the services.

## 3. Ensure Minikube and Docker Are Running

Before proceeding, ensure that Minikube and Docker are running locally:

1. **Start Minikube:**

   ```bash
   minikube start
   ```

2. **Ensure Docker is running** to build and push Docker images.

## 4. Set Up NFS Provisioner

NFS is used for persistent storage in Kubernetes. To set it up:

1. **Create the NFS namespace:**

   ```bash
   kubectl create namespace nfs
   ```

2. **Add the NFS repository:**

   ```bash
   helm repo add nfs-ganesha-server-and-external-provisioner https://kubernetes-sigs.github.io/nfs-ganesha-server-and-external-provisioner
   ```

3. **Install the NFS provisioner:**

   ```bash
   helm upgrade --install -n nfs in-cluster nfs-ganesha-server-and-external-provisioner/nfs-server-provisioner    --set 'storageClass.mountOptions={vers=4.1}'    --set persistence.enabled=true    --set persistence.size=8Gi
   ```

## 5. Create the ERPNext Namespace

To create the namespace for ERPNext:

```bash
kubectl create namespace erpnext
```

Now, install the Frappe/ERPNext release in the `erpnext` namespace using Helm:

```bash
helm install frappe-bench ./erpnext -n erpnext --set persistence.worker.storageClass=nfs
```

To check the status of your Helm release, run:

```bash
helm status frappe-bench -n erpnext
```

## 6. Set the Administrator Password (if needed)

If you need to set the administrator password for the Frappe instance, do the following:

1. **Access the worker pod:**

   ```bash
   kubectl exec -it -n erpnext <frappe-bench-erpnext-worker-s-id> -- bash
   ```

2. **Set the administrator password:**

   ```bash
   bench --site erp.cluster.local set-admin-password admin
   ```

3. **Exit the pod:**

   ```bash
   exit
   ```

## 7. Port Forwarding and Accessing ERPNext

1. **Edit your `hosts` file to point `erp.cluster.local` to `127.0.0.1`:**

   Add the following line to your system's `/etc/hosts` or `C:\Windows\System32\drivers\etc\hosts` file:

   ```
   127.0.0.1   erp.cluster.local
   ```

2. **Forward the port to make ERPNext accessible:**

   ```bash
   kubectl port-forward svc/frappe-bench-erpnext 8081:8080 -n erpnext
   ```

3. **Access ERPNext:**

   Open your browser and visit:

   ```
   http://erp.cluster.local:8081/app/home
   ```

---

This completes the deployment of Frappe/ERPNext using Kubernetes, Docker, NFS, and Helm. You can now access your ERPNext instance and manage it accordingly.
