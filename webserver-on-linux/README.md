# Install a webserver on Linux in 15 minutes
Let's install a containerized web server stack on Linux! That will also deploy an intrusion prevention system with fail2ban, obtain trusted https certificates and a database server to install a WordPress blog.

We will use the free and open-source software Nginx, Docker, and WordPress.

**Nginx**:
Project Homepage: 
Documentation: 

**Docker**:
Project Homepage: 
Documentation: 

**WordPress**:
Project Homepage: 
Documentation: 

Video: https://youtu.be/7GTYB8RVYBc

## 0. Prerequisites

- Linux Server running Ubuntu 20.04 LTS or newer
- Domain that points to the public IP of your Linux Server

You can still install Docker on a Linux Server that is not running Ubuntu, however, this may require different commands!

## 1. Install Docker, and Docker-Compose

You can still install Docker on a Linux Server that is not running Ubuntu, however, this may require different commands!

### 1.1. Install Docker
```bash
sudo apt update

sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt update

sudo apt-get install docker-ce docker-ce-cli containerd.io
```

### 1.2. Check if Docker is installed correctly
```bash
sudo docker run hello-world
```

### 1.3. Install Docker-Compose

Download the latest version (in this case it is 1.25.5, this may change whenever you read this tutorial!)

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
```

### 1.4. Check if Docker-Compose is installed correctly
```bash
sudo docker-compose --version
```

### 1.5. (optional) Add your linux user to the `docker` group
```bash
sudo usermod -aG docker $USER
```

### 1.6. Create a new Docker-Compose file

Create a new folder in the `/opt` directory.

You can also use your personal home folder `/home/<your-username>`, this may require different permissions.

## 2. Set up an Nginx Webserver

### 2.1. Create a new file `docker-compose.yml`file, please refer to the linuxserver/swag documentation: https://hub.docker.com/r/linuxserver/swag.

**Example Docker-Compose File**:
```yml
version: "2"
services:

  swag:
    image: linuxserver/swag
    container_name: swag
    cap_add:
      - NET_ADMIN
    environment:
      - PUID=1001
      - PGID=1001
      - TZ=Europe/Berlin
      - URL=do-test-1.the-digital-life.com
      - SUBDOMAINS=www
      - VALIDATION=http
    volumes:
      - /opt/webserver_swag/config:/config
    ports:
      - 443:443
      - 80:80 #optional
    restart: unless-stopped
```

### 2.2. To start your webserver just execute the following command, while you're in the /opt/webserver_swag folder.

```bash
docker-compose up -d
```

## 3. Add a database to the webserver

### 3.1. Update the Docker-Compose file

You can update your existing Docker-Compose template, if you want to add a database to your webserver. Because you often need a MySQL database, for example to deploy a WordPress Blog.

**Example Docker-Compose File**:
```yml
version: "2"
services:

  mariadb:
    image: linuxserver/mariadb
    container_name: mariadb
    environment:
      - PUID=1001
      - PGID=1001
      - MYSQL_ROOT_PASSWORD=mariadbpassword
      - TZ=Europe/Berlin
      - MYSQL_DATABASE=WP_database
      - MYSQL_USER=WP_dbuser
      - MYSQL_PASSWORD=WP_dbpassword
    volumes:
      - /opt/webserver_swag/config/mariadb:/config
    restart: unless-stopped

  swag:
    image: linuxserver/swag
    container_name: swag
    cap_add:
      - NET_ADMIN
    environment:
      - PUID=1001
      - PGID=1001
      - TZ=Europe/Berlin
      - URL=do-test-1.the-digital-life.com
      - SUBDOMAINS=
      - VALIDATION=http
    volumes:
      - /opt/webserver_swag/config:/config
    ports:
      - 443:443
      - 80:80 #optional
    depends_on:
      - mariadb
    restart: unless-stopped
```