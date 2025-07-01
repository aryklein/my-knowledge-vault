---
tags:
  - containers
  - linux
---

# Podman Cheatsheet

## Image Management

### Pulling Images
```bash
# Pull from Docker Hub
podman pull docker.io/library/postgres
podman pull nginx
podman pull ubuntu:22.04

# Pull from specific registry
podman pull quay.io/podman/stable
```

### Listing Images
```bash
# List all images
podman images

# List images with specific format
podman images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

### Removing Images
```bash
# Remove specific image
podman rmi nginx

# Remove all unused images
podman image prune

# Force remove image
podman rmi -f image_id
```

## Container Management

### Running Containers
```bash
# Run container interactively
podman run -it ubuntu:22.04 /bin/bash

# Run container in background
podman run -d --name webserver -p 8080:80 nginx

# Run with environment variables
podman run -e POSTGRES_PASSWORD=mypass -d postgres

# Run with volume mount
podman run -v /host/path:/container/path nginx
```

### Container Operations
```bash
# List running containers
podman ps

# List all containers (including stopped)
podman ps -a

# Stop container
podman stop container_name

# Start stopped container
podman start container_name

# Restart container
podman restart container_name

# Remove container
podman rm container_name

# Execute command in running container
podman exec -it container_name /bin/bash
```

### Container Logs and Monitoring
```bash
# View container logs
podman logs container_name

# Follow logs in real-time
podman logs -f container_name

# Show container resource usage
podman stats

# Inspect container details
podman inspect container_name
```

## Pod Management

### Creating and Managing Pods
```bash
# Create a pod
podman pod create --name mypod -p 8080:80

# Add container to pod
podman run -dt --pod mypod nginx

# List pods
podman pod ps

# Stop pod
podman pod stop mypod

# Remove pod
podman pod rm mypod
```

## Network Management

### Network Operations
```bash
# List networks
podman network ls

# Create custom network
podman network create mynetwork

# Run container on specific network
podman run --network mynetwork nginx

# Remove network
podman network rm mynetwork
```

## Volume Management

### Volume Operations
```bash
# Create volume
podman volume create myvolume

# List volumes
podman volume ls

# Inspect volume
podman volume inspect myvolume

# Remove volume
podman volume rm myvolume

# Remove all unused volumes
podman volume prune
```

## Registry Configuration

### Configure Registries
By default, no container image registries are configured in Arch Linux. This means 
unqualified searches like podman search httpd will not work. To make Podman behave like
Docker, create a file called 
`/etc/containers/registries.conf.d/10-unqualified-search-registries.conf` with:

```toml
unqualified-search-registries = ["docker.io"]
```

## System Management

### System Information and Cleanup
```bash
# Show system information
podman system info

# Show disk usage
podman system df

# Clean up everything (containers, images, volumes, networks)
podman system prune -a

# Clean up with force (no confirmation)
podman system prune -a -f
```

## Podman Compose

### Using Podman with Docker Compose
```bash
# Install podman-compose
pip install podman-compose

# Run compose file
podman-compose up -d

# Stop compose services
podman-compose down
```

## Rootless Mode

### Running Podman as Non-root User
```bash
# Enable user namespaces (if needed)
echo 'user.max_user_namespaces=28633' | sudo tee -a /etc/sysctl.conf

# Start user services
systemctl --user enable podman.socket
systemctl --user start podman.socket

# Check rootless setup
podman system info --format json | jq .host.security.rootless
```
