Here are the detailed steps to create an Ubuntu-based Docker image with Nginx, a custom `index.html`, build it, run it locally, push it to Docker Hub, and test it using `curl -I`:

### Step 1: Create Dockerfile 

#### Dockerfile

```dockerfile
mkdir ~/docker-example
cat <<EOF > Dockerfile
# Use an official Ubuntu image as the base
FROM ubuntu:latest

# Install necessary packages
RUN apt-get update && apt-get install -y \
    nginx \
    curl \
    vim

# Set up NGINX configuration
RUN echo "daemon off;" >> /etc/nginx/nginx.conf

# Copy custom HTML file
RUN echo "<html><body><h1>Hello from Docker!</h1></body></html>" > /var/www/html/index.html

# Expose port 80
EXPOSE 80

# Start NGINX
CMD ["nginx"]
EOF
```



### Step 2: Build the Docker Image

Navigate to the directory containing your `Dockerfile` and `index.html`, then build the Docker image:

```bash
docker build -t my-nginx-image .
```

### Step 3: Run the Docker Container Locally

After building the image, run it locally to verify it works: ( docker ps -a , docker stop , docker rm , docker exec -it )

```bash
docker run -d --name my-container -p 8080:80 my-nginx-image
```

### Step 4: Test the Container using curl -I

Use `curl -I` to test if the server responds correctly:

```bash
curl -I http://localhost:8080
```

### Step 5: Push the Docker Image to Docker Hub

Assuming you have a Docker Hub account and a repository created:

#### Login to Docker Hub (if not already logged in)

```bash
docker login
```

#### Tag the Docker Image

Tag your local Docker image with your Docker Hub username and repository name:

```bash
docker tag my-nginx-image sasebastian/my-nginx-image:latest
```

Replace `your-docker-hub-username` with your actual Docker Hub username.

#### Push the Docker Image to Docker Hub

Push the tagged image to Docker Hub:

```bash
docker push sasebastian/my-nginx-image:latest
```

### Step 6: Test the Pushed Docker Image

Pull the image from Docker Hub to test if it deploys correctly:

```bash
docker pull sasebastian/my-nginx-image:latest
docker run -d --name my-container -p 8080:80 sasebastian/my-nginx-image:latest
```

### Step 7: Verify with curl -I

Once the container from Docker Hub is running locally, verify again using `curl -I`:

```bash
curl -I http://localhost:8080
```

### Summary

These steps guide you through creating a Docker image with Nginx and a custom `index.html`, testing it locally, pushing it to Docker Hub, pulling and testing it again from Docker Hub, and verifying its functionality using `curl -I`. Adjust paths, names, and content as per your specific requirements.
