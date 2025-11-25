# Application Deployment Guide

This guide explains how to deploy the application using Docker Compose. The application is packaged as a prebuilt Docker image, and all necessary configurations are provided in the `.env` file.

---

## Prerequisites

Before you begin, ensure the following are installed on your system:

1. **Docker**:

   - Install Docker by following the official guide: [Install Docker](https://docs.docker.com/get-docker/).

2. **Docker Compose**:

   - Install Docker Compose by following the official guide: [Install Docker Compose](https://docs.docker.com/compose/install/).

3. **Firewall Configuration**:
   - Ensure port `8080` is open for external access.

---

## Deployment Steps

### Step 1: Prepare the Deployment Directory

1. Create a directory for the deployment files:

   ```bash
   mkdir your-app-deployment
   cd your-app-deployment

   ```

2. Copy the following files into the directory:
   - docker-compose.yml (provided in this repository)
   - env.example (provided in this repository)

### Step 2: Environment Variables

   You need to fill all the environmental variables that missing in the env.example file and create a .env file.

   The application uses the following environment variables, which are defined in the .env file:

   ```
   DATABASE_NAME = connector
   DATABASE_USER = postgres
   DATABASE_PASSWORD = password # Replace it with a secure password

   MINIO_ACCESS_KEY = minioadmin # Replace it with a secure access key
   MINIO_SECRET_KEY = minioadmin # Replace it with a secure secret key
   MINIO_USE_SSL = false
   BUCKET_NAME = connector # Replace with the name of the bucket you want

   KC_URL = # Suite5 will provide this information
   KC_REALM = # Suite5 will provide this information
   KC_CLIENT_ID = # Suite5 will provide this information
   KC_CLIENT_SECRET = # Suite5 will provide this information

   CLOUD_CATALOG_URL = https://dataspace.airedgio50.s5projects.eu/

   # UI
   FRONTEND_URL = https://your-domain.com # Use the public domain you configure in Nginx
   AUTH_SECRET = "ujUjvqbIHbQnToxV2eM5QqfOPrb5KGEt5p7MVcWvZpk=" # Generate and fill this according to instructions below
   KC_FE_CLIENT_ID = # Suite5 will provide this information
   ```

   Ensure the .env file is in the same directory as the docker-compose.yml file.

   To generate a secret for `AUTH_SECRET` environmental variable you can use this command
   ```bash 
   openssl rand -base64 32 
   ``` 
   in your terminal, copy the output and paste it to the appropriate variable in your .env

   For the variables that have a comment to replace them please use something secure, if not the application will deployed properly but with default values.
   When you add all the variables you need copy env.example and create the .env file using the following command
   Linux/MacOs
   ```bash
   cp env.example .env
   ```
   Windows
   ```bash
   copy env.example .env
   ```

### Step 3:  Run the Application

Start the application using Docker Compose:

```bash
docker-compose up -d
```

### Step 4: Public Access & Reverse Proxy
By default, the application listens on port 8080 (HTTP) and is only accessible on your server’s local network.
For production, it is strongly recommended to set up a reverse proxy (such as Nginx) to:

- Expose your app on standard HTTP(S) ports (80/443)

- Serve the application over HTTPS with a custom domain or public IP

- Add security features such as HTTPS, rate limiting, or request logging

Example: Nginx Reverse Proxy
Install Nginx:

```bash
sudo apt update
sudo apt install nginx
```
Create an Nginx configuration (replace your-domain.com with your domain or your server’s public IP):

```bash
server {
    listen 80;
    server_name your-domain.com;
    proxy_buffer_size     32k;
    proxy_buffers       8 32k;
    proxy_busy_buffers_size  64k;
    large_client_header_buffers 4 32k;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Enable configuration (example for Ubuntu/Debian):

```bash
sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

Now your application will be accessible via http://your-domain.com (or your server’s IP).

Securing the Application with HTTPS
To enable HTTPS for your domain, you may use Let’s Encrypt:

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

### Step 5: Access the Application

Open a browser and navigate to: http://your-server-ip/your-url
Replace your-server-ip/your-url with your server's IP address or localhost if running locally.
After this you need to contact Suite5 for keycloak credentials.


### Stopping the Application

To stop the application, run:

```bash
docker-compose down
```
