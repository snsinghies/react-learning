
# Deploying a React App in Kubernetes

## Step 1: Build the React Application
First, build the static files for your React app:

```bash
npm run build
```
This command will create a `build/` directory containing the compiled static files (HTML, CSS, JS) that can be served by a web server.

## Step 2: Create a Dockerfile
To containerize the React app, create a `Dockerfile` in the root of your project directory. This Dockerfile will set up a web server (e.g., Nginx) to serve your static React app.

Here’s an example `Dockerfile`:

```Dockerfile
# Step 1: Build the react app
FROM node:14 AS build

WORKDIR /app

COPY package.json /app/
COPY package-lock.json /app/
RUN npm install

COPY . /app

RUN npm run build

# Step 2: Serve the build with Nginx
FROM nginx:alpine

COPY --from=build /app/build /usr/share/nginx/html

# Copy the Nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf

# Expose port 80
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

## Step 3: Create Nginx Configuration (Optional)
You can optionally configure Nginx to serve your app by creating an `nginx.conf` file. Here’s a basic example:

```nginx
server {
    listen 80;
    server_name localhost;

    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri /index.html;
    }
}
```

This configuration ensures that your React app can handle client-side routing properly.

## Step 4: Build and Tag the Docker Image
Build and tag your Docker image:

```bash
docker build -t my-react-app:latest .
```

## Step 5: Push the Docker Image to a Container Registry
Push your Docker image to a container registry like Docker Hub, Amazon ECR, or Google Container Registry. For Docker Hub, the commands would be:

```bash
docker tag my-react-app:latest <your-dockerhub-username>/my-react-app:latest
docker push <your-dockerhub-username>/my-react-app:latest
```

## Step 6: Create Kubernetes Deployment and Service YAML

Create a `deployment.yaml` and a `service.yaml` file for your Kubernetes setup.

### deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-ui-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: react-ui
  template:
    metadata:
      labels:
        app: react-ui
    spec:
      containers:
      - name: react-ui
        image: <your-dockerhub-username>/my-react-app:latest
        ports:
        - containerPort: 80
```

### service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: react-ui-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: react-ui
```

## Step 7: Apply the Deployment and Service Files
Deploy the React app to your Kubernetes cluster using `kubectl`:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

## Step 8: Access the React Application
If you are using a cloud Kubernetes service (e.g., AWS EKS, GCP GKE), the `Service` with `LoadBalancer` type will expose your application via an external IP.

Run the following command to check the external IP:

```bash
kubectl get services
```

You should see the `EXTERNAL-IP` column populated. Use that IP in your browser to access the deployed React app.

---

That’s it! You have now successfully deployed a React-based UI on Kubernetes.
