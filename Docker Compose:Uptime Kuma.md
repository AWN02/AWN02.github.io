# Docker Compose: Uptime Kuma
## Install Docker on Ubuntu:
Instructions on: https://docs.docker.com/engine/install/
Uninstall old versions:
```
sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc | cut -f1) 
```
* Removes old Docker packages

Install using apt repository:
```
sudo apt update
```
* Updates package list
```
sudo apt install ca-certificates curl
```
* Installs certificates and curl so it can download Docker files safely
```
sudo install -m 0755 -d /etc/apt/keyrings
```
* Creates directory to store security key
```
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
```
* Downloads Docker GPG key
```
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
* Gives permission to read the key
```
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
```
Paste:
```
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```
* Adds docker repository
```
sudo apt update
```
* Updates package list
## Install Docker Compose
```
sudo apt update
```
```
sudo apt install docker-compose-plugin
```
* Installs docker compose command
Test with:
```
docker compose version
```
* Gives version of docker compose (to test if it works)
## Install Uptime Kuma
```
mkdir -p ~/uptime-kuma && cd ~/uptime-kuma
```
* Creates directory for Uptime Kuma
```
sudo nano docker-compose.yml
```
Paste:
```
version: "3.8"

services:
  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    restart: always
    ports:
      - "3001:3001"  # This maps the container port "3001" to the host port "3001"
    volumes:
      - /path/to/data:/app/data  # Configuring persistent storage
    environment:
      - TZ=UTC  # Set the timezone (change to your preferred local timezone so monitoring times are the same)
      - UMASK=0022  # Set your file permissions manually
    networks:
      - kuma_network  # add your own custom network config
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3001"]
      interval: 30s
      retries: 3
      start_period: 10s
      timeout: 5s
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

networks:
  kuma_network:
    driver: bridge
```
Hit Ctrl+O to write to file and Ctrl+X to exit

* Creates the docker-compose.yml file
```
docker-compose up -d
```
* Starts Uptime Kuma container

In a web browser, paste the following:
```
http://<your-server-ip-here>:3001
```
In my case, I paste: 
```
http://10.30.83.10:3001
```
