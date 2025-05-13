# Multi-Container VM Setup with Traefik

This project demonstrates a multi-container setup on Google Compute Engine using Traefik as a reverse proxy.

## Prerequisites

- Google Cloud Platform account
- Google Cloud SDK installed
- Docker and Docker Compose installed on your local machine

## Setup Instructions

1. Create a new VM instance on Google Compute Engine:
   ```bash
   gcloud compute instances create traefik-vm \
     --machine-type=e2-micro \
     --zone=europe-north2-a \
     --image-family=ubuntu-minimal-2404-lts-amd64  \
     --image-project=ubuntu-os-cloud \
     --tags=http-server,https-server
   ```

2. SSH into your VM:
   ```bash
   gcloud compute ssh traefik-vm
   ```

3. Install Docker and Docker Compose on the VM:
   ```bash
   sudo apt-get update
   sudo apt-get install -y docker.io docker-compose
   sudo add-apt-repository universe && sudo apt-get update
   sudo apt install python3-setuptools
   sudo usermod -aG docker $USER
   ```

4. Clone this repository to your VM:
   ```bash
   git clone <your-repo-url>
   cd <repo-directory>
   ```

5. Start the containers:
   ```bash
   docker-compose up -d
   ```

## SSL Configuration

The setup uses Let's Encrypt for automatic SSL certificate generation. To configure SSL:

1. Replace `your-email@example.com` in both `traefik.yml` and `docker-compose.yml` with your actual email address
2. Replace `yourdomain.com` in the docker-compose.yml with your actual domain name
3. Ensure your domain's DNS records point to your VM's IP address
   - Add an A record for your domain:
     - Name: @ (or leave blank, depending on your DNS provider)
     - Value: Your VM's IP address
     - TTL: 3600 (or default)
   - Add A records for each subdomain:
     - Name: traefik
     - Value: Your VM's IP address
     - TTL: 3600 (or default)
     
     - Name: whoami
     - Value: Your VM's IP address
     - TTL: 3600 (or default)
     
     - Name: hello
     - Value: Your VM's IP address
     - TTL: 3600 (or default)
   
   Note: You can find your VM's IP address using:
   ```bash
   gcloud compute instances describe traefik-vm --zone=europe-north2-a --format="get(networkInterfaces[0].accessConfigs[0].natIP)"
   ```
4. Make sure ports 80 and 443 are open in your firewall
5. Create and set proper permissions for acme.json:
   ```bash
   touch acme.json
   chmod 600 acme.json
   ```
   This file is used by Let's Encrypt to store certificates and must have restricted permissions (600) to work properly.

The following subdomains will be available with SSL:
- `traefik.yourdomain.com` - Traefik dashboard
- `whoami.yourdomain.com` - Whoami service
- `hello.yourdomain.com` - Python Hello World application

## Network Configuration

To ensure proper network access, you need to configure the following firewall rules on your Google Compute Engine instance:

1. Create a firewall rule for HTTP (port 80):
   ```bash
   gcloud compute firewall-rules create allow-http \
       --direction=INGRESS \
       --priority=1000 \
       --network=default \
       --action=ALLOW \
       --rules=tcp:80 \
       --source-ranges=0.0.0.0/0 \
       --target-tags=http-server
   ```

2. Create a firewall rule for HTTPS (port 443):
   ```bash
   gcloud compute firewall-rules create allow-https \
       --direction=INGRESS \
       --priority=1000 \
       --network=default \
       --action=ALLOW \
       --rules=tcp:443 \
       --source-ranges=0.0.0.0/0 \
       --target-tags=https-server
   ```

3. Create a firewall rule for Traefik dashboard (port 8080):
   ```bash
   gcloud compute firewall-rules create allow-traefik-dashboard \
       --direction=INGRESS \
       --priority=1000 \
       --network=default \
       --action=ALLOW \
       --rules=tcp:8080 \
       --source-ranges=0.0.0.0/0 \
       --target-tags=http-server
   ```

These rules will allow:
- HTTP traffic on port 80
- HTTPS traffic on port 443
- Traefik dashboard access on port 8080

Make sure your VM instance has the correct network tags:
- `http-server`
- `https-server`

You can verify the network tags with:
```bash
gcloud compute instances describe traefik-vm --zone=europe-north2-a --format="get(tags.items)"
```

## Accessing the Services

- Traefik Dashboard: https://traefik.yourdomain.com
- Sample Application (Whoami): https://whoami.yourdomain.com
- Python Hello World App: https://hello.yourdomain.com

## Services

The setup includes three services:
1. Traefik - Reverse proxy and load balancer
2. Whoami - Sample application for testing
3. Hello World - Python Flask application

## Configuration

- `docker-compose.yml`: Contains the container definitions and networking setup
- `traefik.yml`: Traefik configuration file
- `hello_world/`: Directory containing the Python Flask application
  - `app.py`: Main application file
  - `Dockerfile`: Container definition
  - `requirements.txt`: Python dependencies

## Security Notes

- This setup uses insecure mode for demonstration purposes
- For production use, please:
  - Enable HTTPS
  - Configure proper authentication
  - Use secure configuration for Traefik
  - Set up proper firewall rules

## Security Considerations

### Development vs Production
- The current configuration uses `insecure: true` in Traefik for development purposes
- For production deployment:
  - Disable insecure mode
  - Configure proper SSL/TLS certificates
  - Set up authentication for the Traefik dashboard
  - Use environment variables for sensitive configuration

### Docker Socket Access
- The setup mounts the Docker socket (`/var/run/docker.sock`) for Traefik
- This allows Traefik to discover containers automatically
- In production, consider:
  - Using a more restricted mount
  - Implementing proper access controls
  - Using Docker API instead of socket mount

### Network Security
- The setup exposes ports 80 and 8080
- Ensure proper firewall rules are in place
- Consider using a VPN or private network for the Traefik dashboard
- Implement rate limiting and other security measures

## Troubleshooting

1. Check container status:
   ```bash
   docker-compose ps
   ```

2. View container logs:
   ```bash
   docker-compose logs
   ```

3. Ensure firewall rules allow traffic on ports 80 and 8080 

4. Checking SSL Status:
   ```bash
   # Check SSL certificate generation status
   docker-compose logs -f traefik | grep -i "acme\|cert\|challenge"
   
   # If you see permission errors, fix acme.json permissions:
   chmod 600 acme.json
   
   # Restart Traefik after fixing permissions:
   docker-compose restart traefik
   ``` 