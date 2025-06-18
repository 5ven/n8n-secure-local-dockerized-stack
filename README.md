# Secure n8n Deployment with Docker Compose & Nginx

This guide documents the process for deploying a secure, persistent n8n instance on a local network server. The stack is managed via Docker Compose and includes:

* **n8n:** The core workflow automation service.
* **Nginx:** A reverse proxy to handle SSL/TLS termination.
* **PostgreSQL:** A robust database backend for n8n.
* **mkcert:** For generating locally trusted SSL certificates.

Configuration is managed via a `.env` file to keep secrets out of version control. The setup is designed for a headless Linux server (Arch Linux) managed from a macOS client.

## Prerequisites

### On the Arch Linux Server

* `docker`
* `docker-compose`
* `git` (optional, for version control)


### On the macOS Client

* `homebrew`
* `mkcert`


## 1. Local Certificate Generation (on macOS)

We will generate certificates on the macOS client, signed by a local Certificate Authority (CA) that is automatically trusted by your browsers.

1. **Install mkcert on your Mac:**

```bash
brew install mkcert
```

2. **Create and install a local CA:**
This is a one-time step. It installs a root certificate into your system and browser trust stores.

```bash
mkcert -install
```

3. **Generate the server certificate:**
Create a certificate that is valid for the hostname and IP address of your Arch server. Replace `your-hostname` and `your.server.ip` accordingly.

```bash
# Example: mkcert -key-file n8n.key -cert-file n8n.crt n8n 192.168.1.67
mkcert -key-file n8n.key -cert-file n8n.crt <your-hostname> <your.server.ip>
```

This will create `n8n.key` and `n8n.crt` files in your current directory.

## 2. Server Setup (on Arch Linux)

All subsequent steps are performed on the Arch server.

1. **Prepare the project directory:**
Clone this repository or create the directory structure manually.

```bash
mkdir -p ~/n8n/certs ~/n8n/nginx
cd ~/n8n
```

2. **Copy certificates from your Mac to the server:**
Run this command from your Mac's terminal, in the directory where you generated the certificates.

```bash
# Example: scp n8n.key n8n.crt sven@192.168.1.67:~/n8n/certs/
scp n8n.key n8n.crt <user>@<your.server.ip>:~/n8n/certs/
```

3. **Create the environment file (`.env`):**
Create a file named `.env` in the project root. This file stores your configuration and secrets.

```bash
# ~/n8n/.env

# --- PostgreSQL Credentials ---
POSTGRES_DB=n8n
POSTGRES_USER=n8n_user
POSTGRES_PASSWORD=YOUR_SUPER_SECRET_PASSWORD

# --- n8n Configuration ---
# The public-facing hostname for n8n
N8N_HOST=n8n
GENERIC_TIMEZONE=Europe/London
N8N_EDITOR_BASE_URL=https://thebaseurl.domain
WEBHOOK_URL=https://thebaseurl.domain

# --- Docker Permissions ---
# User and Group ID for the n8n container to avoid permission issues
PUID=1000
PGID=1000
```

## 3. Client-Side Host Configuration (on macOS)

Teach your Mac where to find your new n8n instance.

1. Edit your hosts file:

```bash
sudo nano /etc/hosts
```

2. Add the following line, then save and exit:

```
# Example: 192.168.1.67   n8n
<your.server.ip>   <your-hostname>
```

## 4. Deployment \& Management

1. **Launch the stack:**
From the `~/n8n` directory on your server, run:

```bash
sudo docker-compose up -d
```

2. **Access your n8n instance:**
Open a browser on your Mac and navigate to `https://<your-hostname>`. You should see a secure connection.
3. **Management Commands:**
    * **View logs:** `sudo docker-compose logs -f`
    * **Stop the stack:** `sudo docker-compose down`
    * **Update the stack:** `sudo docker-compose pull && sudo docker-compose up -d`
