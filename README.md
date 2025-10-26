#  NGINX Blue-Green Deployment Setup (Docker Compose)

This repository demonstrates a **simple blue-green deployment** pattern using **NGINX** as a reverse proxy and **Docker Compose** to manage two versions (Blue and Green) of the same application.  

The setup ensures **zero downtime** during updates and supports fast rollback between releases.



##  Overview

The architecture uses:

- **NGINX** as a load balancer and traffic router.
- **Two backend containers** (`app_blue` and `app_green`) running different versions of the same app.
- **Environment variables** to dynamically switch the active backend.
- **Docker Compose** for easy orchestration of services.


Traffic flows like this:

User → NGINX → (Blue or Green App)


When a new version is ready, NGINX configuration can be updated to switch traffic to the new app (Blue → Green or vice versa) **without downtime**.



## ⚙️ Project Structure


- **├──** docker-compose.yml
- **├──** nginx.conf
- **├──** templates/
- **│ └──** upstreams.conf.template
- **├──** .env
- **└──** README.md



## 🧰 Prerequisites

Before you begin, ensure you have the following installed:

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/)
- Basic understanding of NGINX and container networking


## 🪜 Step-by-Step Setup

### 1️ Clone the Repository

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```


2. ## Configure Environment Variables
Create a .env file in the project root (if it doesn't exist) and define:

BLUE_IMAGE=yimikaade/wonderful:devops-stage-two
GREEN_IMAGE=yimikaade/wonderful:devops-stage-two
ACTIVE_POOL=blue
RELEASE_ID_BLUE=blue-release-v1
RELEASE_ID_GREEN=green-release-v1


  This controls which container is active (ACTIVE_POOL=blue or ACTIVE_POOL=green).

4. ## Set Up the NGINX Configuration
The NGINX configuration is composed of:
nginx.conf

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/conf.d/upstreams.conf;

    proxy_connect_timeout 2s;
    proxy_send_timeout 5s;
    proxy_read_timeout 5s;

    proxy_pass_header Server;
    proxy_hide_header X-Powered-By;

    server {
        listen 8080;

        location / {
            proxy_pass http://active_backend;

            proxy_next_upstream error timeout http_500 http_502 http_503 http_504;
            proxy_next_upstream_timeout 10s;
            proxy_next_upstream_tries 2;

            proxy_pass_request_headers on;
        }
    }
}


# Build and Start Services

Run the following command:
docker-compose up -d

This will start three containers:

nginx (reverse proxy)

app_blue (active backend)

app_green (standby backend)

You can verify the running containers with:

docker ps
<img width="1182" height="128" alt="image" src="https://github.com/user-attachments/assets/2a4f2907-c4b8-44a5-91f4-4367c705c69f" />




