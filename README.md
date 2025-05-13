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
4. Make sure ports 80 and 443 are open in your firewall

The following subdomains will be available with SSL:
- `traefik.yourdomain.com` - Traefik dashboard
- `whoami.yourdomain.com` - Whoami service
- `hello.yourdomain.com` - Python Hello World application

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