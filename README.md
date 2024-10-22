
# Executing `godotenv` Locally and in Kubernetes

To execute your application both locally and in Kubernetes, follow these steps:

## Local Development Setup

1. **Install Go and `godotenv`**:  
   Ensure you have Go installed locally. You can install the `godotenv` package to load environment variables from the `.env` file.
   ```bash
   go get github.com/joho/godotenv
   ```

2. **Create a `.env` File**:  
   Define your environment variables in a `.env` file for local use:
   ```dotenv
   DB_HOST=localhost
   DB_PORT=5432
   DB_USER=user
   DB_PASSWORD=password
   ```

3. **Use `godotenv` in Your Go Code**:  
   Load environment variables from the `.env` file in your Go code.
   ```go
   package main

   import (
       "github.com/joho/godotenv"
       "log"
       "os"
   )

   func main() {
       err := godotenv.Load()
       if err != nil {
           log.Fatalf("Error loading .env file")
       }

       dbHost := os.Getenv("DB_HOST")
       log.Println("DB Host:", dbHost)
   }
   ```

4. **Run Your Go Application**:  
   Execute the application in your local environment.
   ```bash
   go run main.go
   ```

## Running in Kubernetes

1. **Prepare a Docker Image**:  
   To run your application in Kubernetes, you first need to containerize it using Docker.  
   Create a `Dockerfile`:
   ```Dockerfile
   FROM golang:1.21-alpine

   WORKDIR /app
   COPY . .

   RUN go mod tidy
   RUN go build -o main .

   CMD ["./main"]
   ```

2. **Build and tag the Docker image**:
   ```bash
   docker build -t your-app-image .
   ```

3. **Push the Docker Image to a Container Registry**:  
   Push your image to a registry like Docker Hub or Google Container Registry (GCR).
   ```bash
   docker tag your-app-image your-dockerhub-username/your-app-image
   docker push your-dockerhub-username/your-app-image
   ```

4. **Kubernetes ConfigMap for Environment Variables**:  
   In Kubernetes, use a ConfigMap to store environment variables. You can create this YAML file for non-sensitive data.  
   `configmap.yaml`:
   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: app-config
   data:
     DB_HOST: "db-prod.example.com"
     DB_PORT: "5432"
     DB_USER: "produser"
   ```

5. **Create a Kubernetes Secret for Sensitive Data**:  
   For sensitive data like passwords, use a Secret.  
   `secret.yaml`:
   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: app-secrets
   type: Opaque
   data:
     DB_PASSWORD: cHJvZHBhc3N3b3Jk  # Base64 encoded
   ```

6. **Kubernetes Deployment**:  
   Create a Deployment YAML to run your app in Kubernetes.  
   `deployment.yaml`:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: your-app
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: your-app
     template:
       metadata:
         labels:
           app: your-app
       spec:
         containers:
           - name: your-app
             image: your-dockerhub-username/your-app-image
             envFrom:
               - configMapRef:
                   name: app-config
               - secretRef:
                   name: app-secrets
   ```

7. **Deploy to Kubernetes**:  
   Apply the ConfigMap, Secret, and Deployment YAMLs to your Kubernetes cluster.
   ```bash
   kubectl apply -f configmap.yaml
   kubectl apply -f secret.yaml
   kubectl apply -f deployment.yaml
   ```

8. **Verify the Application is Running**:  
   Check if your pods are running:
   ```bash
   kubectl get pods
   ```

9. **Access Logs (optional)**:  
   View logs to ensure the application is functioning correctly.
   ```bash
   kubectl logs <your-pod-name>
   ```

This setup allows you to run your application locally using `.env` with `godotenv` and deploy it in Kubernetes, where it reads environment variables from ConfigMaps and Secrets.
