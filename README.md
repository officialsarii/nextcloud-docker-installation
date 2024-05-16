# nextcloud-docker-installation
installing your nextcloud in docker

In a nutshell, Nextcloud is an open-source platform that provides file sharing, collaboration, and office suite functionality similar to popular services like Google Drive or Office 365. You can install it on your own server for maximum control over your data, and this guide walks you through setting it up using Docker for easier management.


**Prerequisites**

- A running Ubuntu server
- Root or sudo privileges

**Step-by-Step Installation Guide: Nextcloud on Docker**

This guide details how to install Nextcloud, a self-hosted file sharing and collaboration platform, on Ubuntu using Docker for easier management.

**1. Update and Upgrade System Packages:**

```bash
sudo apt-get update && sudo apt-get upgrade -y
```
Explanation:
sudo: Grants root privileges for package management.
apt-get: Package manager for Ubuntu.
update: Fetches the latest package lists from repositories.
upgrade: Upgrades installed packages to their latest versions.
-y: Option for automatic "yes" to prompts during upgrades.


2. Install Docker Engine:
```Bash
sudo apt-get install docker.io
```
Explanation:
docker.io: The official Docker engine repository.
This installs the necessary components to run Docker containers.

3. Install Docker Compose:
```Bash
sudo apt-get install docker-compose-v2
```
Explanation:
docker-compose-v2: Installs the specific version of Docker Compose. This may vary depending on your system's configuration.
Docker Compose is a tool for managing multi-container applications.


4. Set Server Hostname (Optional):
```Bash
sudo nano /etc/hostname
```
Explanation:
sudo nano: Opens the hostname configuration file in the Nano text editor.
Edit the file to include your desired domain name (e.g., yourdomain.com).
This step is optional but recommended for better server management and accessibility.
Save the changes and exit the editor (Ctrl+O, then Enter, followed by Ctrl+X).

5. Grant Docker Access to Your User:
```Bash
usermod -aG docker <username>
```
Explanation:
usermod: Modifies user account settings.
-aG: Adds a user to a group.
docker: The target group for Docker access.
<username>: Replace with your actual username.
This allows your user to manage Docker containers without requiring root privileges.

6. Enable Firewall and Open Ports:
```Bash
sudo ufw enable
sudo ufw allow 22/tcp  # Open SSH port for secure remote access
sudo ufw allow 80/tcp  # Open HTTP port for web traffic
sudo ufw allow 443/tcp # Open HTTPS port for secure web traffic
```
Explanation:
sudo ufw: Manages the firewall settings (ufw stands for "Uncomplicated Firewall").
enable: Enables the firewall.
allow: Allows specific ports through the firewall.
22/tcp: Standard port for SSH (Secure Shell) connections.
80/tcp: Standard port for HTTP (web traffic).
443/tcp: Standard port for HTTPS (secure web traffic).

7. Create Nextcloud Project Directory:
```Bash
mkdir nextcloud && cd nextcloud
```
Explanation:
mkdir: Creates a directory named nextcloud.
cd nextcloud: Changes the working directory to nextcloud.

8. Create a Database Environment File (db.env):
```Bash
sudo nano db.env
```
Explanation:
sudo nano: Opens the db.env file for editing.
Enter the following lines (replace placeholders with your desired values):
```MYSQL_ROOT_PASSWORD=your_strong_root_password
MYSQL_PASSWORD=your_strong_mysql_password
MYSQL_DATABASE=db
MYSQL_USER=your_unique_mysql_username
```
Important:
MYSQL_ROOT_PASSWORD: Use a strong and unique password for the MySQL root user.
MYSQL_PASSWORD: Use a strong and unique password for the MySQL user that Nextcloud will use.
MYSQL_DATABASE: Name of the database used by Nextcloud (default: db).
MYSQL_USER: Choose a unique username for the MySQL user that Nextcloud will use (avoid yourmysqluser).
Save the changes and exit the editor.

9. Create docker-compose.yml File or you can find it on the project:
```Bash
sudo nano docker-compose.yml
```
Explanation:
sudo nano: Opens the docker-compose.yml file for editing.
Paste the following content into the file:

```
# Define the Docker Compose version
version: '3'

# Define services within the application
services:
  # Database service
  db:
    # Use the official mariadb image
    image: mariadb
    # Automatically restart the container unless it's stopped manually
    restart: unless-stopped
    # Set additional configuration options for the database server
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW --innodb-file-per-table=1 --skip-innodb-read-only-compressed
    # Define a persistent volume for the database data
    volumes:
      - db:/var/lib/mysql  # Map volume named 'db' to the container's /var/lib/mysql directory
    # Reference environment variables from the db.env file
    env_file:
      - db.env

  # Redis cache service
  redis:
    # Use the official redis image
    image: redis
    # Always restart the container even if it exits successfully
    restart: always
    # Set the Redis server password (replace with a strong password)
    command: redis-server --requirepass your_redis_password

  # Nextcloud application service
  app:
    # Use the official nextcloud image (latest version)
    image: nextcloud:latest
    # Automatically restart the container unless it's stopped manually
    restart: unless-stopped
    # Map container port 80 (web server) to host port 8080 (adjust if needed)
    ports:
      - 8080:80
    # Link the Nextcloud container to the db and redis services
    links:
      - db
      - redis
    # Define a persistent volume for the Nextcloud application data and configuration
    volumes:
      - nextcloud:/var/www/html  # Map volume named 'nextcloud' to the container's /var/www/html directory
    # Set environment variables for Nextcloud configuration
    environment:
      - MYSQL_HOST=db  # Set the database host address (internal service name)
      - REDIS_HOST_PASSWORD=your_redis_password  # Set the Redis server password (replace with a strong password)
    # Reference environment variables from the db.env file
    env_file:
      - db.env
    # Ensure the db and redis services are started before the app service
    depends_on:
      - db
      - redis

# Define persistent volumes for the application
volumes:
  db:  # Define a volume named 'db' (used by the db service)
  nextcloud:  # Define a volume named 'nextcloud' (used by the app service)


```


10. Run docker-compose up
In the project directory where you have the docker-compose.yml file, run the following command to initiate the deployment of your Nextcloud instance
```
docker-compose up -d
```


## Verifying Nextcloud Container Status and Inspecting Volumes

Once you've completed the previous steps and set up Nextcloud with Docker Compose, you can verify if the containers are running and inspect their associated volumes.

1. Checking Container Status
```
Bash
docker ps
```
Explanation:
docker ps: This command displays a list of all running Docker containers. Look for containers related to nextcloud, db, and redis to confirm they are running.

2. Listing Volumes
```Bash
docker volume ls
```
Explanation:
docker volume ls: This command lists all Docker volumes currently defined on your system. You should see volumes named nextcloud_db and nextcloud_nextcloud created by the Docker Compose setup.

3. Inspecting Volumes
```Bash
docker volume inspect nextcloud_db
docker volume inspect nextcloud_nextcloud
```
Explanation:
docker volume inspect: This command provides detailed information about a specific Docker volume. Here, we're inspecting the nextcloud_db and nextcloud_nextcloud volumes.
The output will show details like the volume driver used, the mount point on the host system (where the volume's data is stored), and any custom configurations associated with the volume.

4. Accessing Nextcloud
With the containers running, you can access the Nextcloud web interface by opening a web browser and navigating to:
http://your_server_IP:8080 (if not using a custom domain name) **you can get your IP by using command ```ifconfig```
https://your_domain (if using a custom domain and SSL certificate)
Note: Replace your_server_IP with your server's actual IP address and your_domain with your configured domain name (if applicable). If using HTTPS with a custom domain, ensure you have a valid SSL certificate installed.
