
# 🛠 Full Command Set to Deploy Go App to Azure Container Apps

---

## 1. 🔹 Set Environment Variables

```bash
ACR_NAME=""
RESOURCE_GROUP=""
LOCATION=""
APP_NAME=""
ENV=""
```

---

## 2. 🔹 Create Resource Group

```bash
az group create --name $RESOURCE_GROUP --location $LOCATION
```

---

## 3. 🔹 Create Azure Container Registry (ACR)

```bash
az acr create --name $ACR_NAME --resource-group $RESOURCE_GROUP --sku Basic --location $LOCATION
```

---

## 4. 🔹 Build Docker Image (Important for Mac M1/M2 Users)

### ⚠️ Special Step for Mac M1 / M2 (ARM64 processors)

Use **Docker Buildx** to build a Linux AMD64 compatible image:

```bash
# Create and use buildx builder
docker buildx create --use

# Build for linux/amd64 and push directly to ACR
docker buildx build --platform linux/amd64 -t $ACR_NAME.azurecr.io/product-service:latest --push .
```

✅ This ensures your container is compatible with Azure Container Apps (linux/amd64 architecture).

---

## 5. 🔹 (If not pushed already) Login to Azure Container Registry

```bash
az acr login --name $ACR_NAME
```

---

## 6. 🔹 Create Azure Container Apps Environment

```bash
az containerapp env create \
  --name $ENV \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION
```

---

## 7. 🔹 Deploy the Container App

```bash
az containerapp create \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --environment $ENV \
  --image $ACR_NAME.azurecr.io/product-service:latest \
  --target-port 8080 \
  --ingress external \
  --registry-server $ACR_NAME.azurecr.io \
  --registry-username "<your-acr-username>" \
  --registry-password "<your-acr-password>"
```

✅ Get your ACR username and password via:

```bash
az acr credential show --name $ACR_NAME
```

Use the `username` and `password` values from the output.

---

## 8. 🔹 Get the Public URL of your Container App

```bash
az containerapp show \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --query properties.configuration.ingress.fqdn \
  --output tsv
```

✅ Your public URL will look like:

```
https://demoappchmaps2025.centralus.azurecontainerapps.io
```

---

# ✅ Quick Access Endpoints:

- `https://your-app-url/health` → Health check
- `https://your-app-url/api/products` → Product APIs
- `https://your-app-url/swagger/index.html` → Swagger UI

---

# ⚡ Quick Notes:

- Always **build using `linux/amd64`** if you're using Mac M1/M2 to avoid OS/Architecture mismatch.
- Azure Container Apps **auto-scales** based on HTTP traffic.
- ACA will **scale to zero** if no traffic = saves cost automatically.

---


# 📌 Azure Container Apps (ACA) – Key Concepts

**Azure Container Apps** is a **serverless** platform built on **Kubernetes** for running microservices and APIs, **without managing Kubernetes clusters**.

✅ Built on Kubernetes and KEDA (Kubernetes Event-Driven Autoscaling)  
✅ Automatic **scale-out** and **scale-in** based on traffic or events  
✅ Supports **HTTP-based APIs**, **event-driven architectures**, and **background workers**  
✅ **Integrated ingress**, scaling, and Dapr (for microservices communication)  
✅ Supports **private container registries (like ACR)** out of the box  
✅ Ideal for **microservices, APIs, background jobs, and event processors**

---

# 🚀 What You Can Do Next

Now that your basic app is deployed, you can explore advanced Azure Container Apps capabilities:

| Feature | Description |
|:--------|:------------|
| 🔥 **Scale to Zero** | Automatically scale down your app to 0 instances when no traffic to save cost. |
| 🌍 **Traffic Splitting** | Roll out new app versions with **canary deployments** (e.g., 80% v1, 20% v2 traffic). |
| 🧹 **Revisions Management** | Deploy new revisions without downtime. |
| ⚡ **Autoscaling (KEDA)** | Scale apps based on queue length, events, CPU usage, etc. |
| 🔒 **Secure Networking** | Integrate with Azure Virtual Network (VNet) for private endpoints. |
| 📈 **Monitoring** | Use Container Apps metrics, Azure Monitor, and distributed tracing. |

---

# 🎯 Example: Traffic Splitting in ACA

Suppose you deploy a new revision and want:

- 90% traffic to **v1** (stable version)
- 10% traffic to **v2** (new version)

You can configure **traffic splitting** like this:

```bash
az containerapp revision set-mode --name $APP_NAME --resource-group $RESOURCE_GROUP --mode multiple

az containerapp ingress traffic set \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --type revision \
  --revision-weight latest=10 stable=90
```

✅ This allows **safe gradual rollout** of new app versions!  
✅ No downtime during upgrades.

---

# 📚 Learn More – Official Microsoft Docs

- 📘 [Azure Container Apps Documentation Overview](https://learn.microsoft.com/en-us/azure/container-apps/overview)
- 📘 [Code to Cloud Options with Azure Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/code-to-cloud-options)
- 📘 [Deploying Revisions and Traffic Splitting](https://learn.microsoft.com/en-us/azure/container-apps/traffic-splitting)
- 📘 [Build and Push Containers with Docker Buildx](https://learn.microsoft.com/en-us/azure/container-apps/build-container-image)
- 📘 [Scale Rules with KEDA](https://learn.microsoft.com/en-us/azure/container-apps/scale-app)

---

# 🏁 Summary

✅ Deployed a fully containerized GoLang Product Service API on Azure Container Apps  
✅ Container built for linux/amd64 using Docker Buildx (important for M1/M2 Macs)  
✅ Image securely stored in Azure Container Registry (ACR)  
✅ App publicly accessible with HTTPS  
✅ Ready for future enhancements like **autoscaling**, **private networking**, **canary deployments**, and **observability**.

---
