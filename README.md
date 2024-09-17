
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
user  nginx;
worker_processes  auto;
error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    server {
        listen       80;
        server_name  localhost;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
    }
}
```

This configuration ensures that your React app can handle client-side routing properly.
#3 NGINX Basic Configuration Explanation

### 1. `user nginx;`
This line specifies the user under which the NGINX server will run. In this case, it’s set to `nginx`. Running the server as the `nginx` user (instead of `root`) is a security measure.

### 2. `worker_processes auto;`
This directive tells NGINX how many worker processes it should use. The `auto` value makes NGINX automatically determine the number of processes based on the available CPU cores. Each worker process handles incoming requests and connections.

### 3. `error_log /var/log/nginx/error.log notice;`
Defines the location and level of logging for errors:
- `/var/log/nginx/error.log` is the path to the error log file.
- `notice` is the logging level, meaning that only messages with a severity level of "notice" or higher will be logged. Other levels include `info`, `warn`, `error`, etc.

### 4. `pid /var/run/nginx.pid;`
Specifies where NGINX should store its process ID (PID). This PID file allows the system or administrator to manage or stop the NGINX process.

### 5. `events { worker_connections 1024; }`
The `events` block defines settings related to handling connections:
- `worker_connections 1024;` means each worker process can handle up to 1024 simultaneous connections. With more worker processes, NGINX can scale accordingly.

### 6. `http { ... }`
Contains all configuration related to HTTP traffic, including serving web content.

### 7. `include /etc/nginx/mime.types;`
Includes the file `/etc/nginx/mime.types`, which maps file extensions to their MIME types (e.g., `.html` for `text/html`, `.png` for `image/png`). This helps NGINX serve the correct content type based on the file being requested.

### 8. `default_type application/octet-stream;`
Sets the default MIME type for files if their type cannot be determined. The value `application/octet-stream` is a generic binary file type, used as a fallback when no other MIME type can be determined.

### 9. `server { ... }`
Defines the settings for a specific virtual server:

### 9.1 `listen 80;`
Tells NGINX to listen on port `80`, which is the default port for HTTP traffic.

### 9.2 `server_name localhost;`
Defines the server’s domain name or IP address. `localhost` means it will handle requests made to `localhost`, typically on the machine itself.

### 9.3 `location / { ... }`
Defines how to handle requests for specific URIs (i.e., web addresses). For the root (`/`):

- `root /usr/share/nginx/html;`: Tells NGINX to serve files from the directory `/usr/share/nginx/html`, where static files (like `index.html`) are located.
- `index index.html index.htm;`: Instructs NGINX to look for `index.html` or `index.htm` files when the user requests the root directory (`/`).

### Summary
- **User and Workers**: The server runs as the `nginx` user and automatically spawns worker processes based on CPU cores.
- **Logging and PID**: Errors are logged at the `notice` level, and the process ID is stored in `/var/run/nginx.pid`.
- **Connection Handling**: Each worker process can handle 1024 simultaneous connections.
- **Serving Content**: NGINX listens on port 80 for requests to `localhost`, serving files from the `/usr/share/nginx/html` directory, with `index.html` as the default file for the root path.

This configuration is suitable for serving static websites (like a React app build) using NGINX.


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
