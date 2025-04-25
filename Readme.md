
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
