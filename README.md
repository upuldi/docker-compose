# Self-Hosted Docker Compose Services

This repository contains a Docker Compose configuration for deploying a variety of self-hosted services, designed to enhance your personal and home lab infrastructure. It provides a robust and scalable solution for managing media, finances, knowledge, logging, and more, all within a Dockerized environment.

## Table of Contents

1.  [Project Overview](#project-overview)
2.  [Home Lab Architecture](#home-lab-architecture)
3.  [Features](#features)
4.  [Prerequisites](#prerequisites)
5.  [Setup Instructions](#setup-instructions)
    * [1. Clone the Repository](#1-clone-the-repository)
    * [2. Configure Environment Variables](#2-configure-environment-variables)
    * [3. Deploy Services](#3-deploy-services)
6.  [Service Breakdown](#service-breakdown)
    * [Core Infrastructure](#core-infrastructure)
    * [Media Management](#media-management)
    * [Financial & Productivity Tools](#financial--productivity-tools)
    * [Logging & Monitoring](#logging--monitoring)
    * [Identity & Access Management](#identity--access-management)
    * [Utilities & Development](#utilities--development)
    * [Optional/Disabled Services](#optionaldisabled-services)
7.  [Network Configuration](#network-configuration)
8.  [Usage and Management](#usage-and-management)
9.  [Troubleshooting](#troubleshooting)
10. [Customization](#customization)

## Project Overview

This project centralizes the deployment and management of various self-hosted applications using Docker Compose. It leverages Docker's containerization to ensure consistency, isolation, and ease of deployment across different environments. The services cover a wide range of functionalities, from media serving and download automation to personal finance, knowledge management, and comprehensive logging/monitoring.

Home Lab Architecture
---------------------

This Docker Compose setup is designed to integrate into a robust home lab environment, distributed across multiple hosts for optimal performance, security, and data management. All self-hosted services are accessed via HTTPS, secured with certificates from Let's Encrypt. Cloudflare is utilized as the external DNS provider, pointing domains to the public IP and leveraging its features.

The architecture is structured as follows:

*   **External Network Layer**    
    
    *   **Internet Traffic** flows through **Cloudflare** (acting as DNS, Proxy, and Firewall).    
    *   **Cloudflare** directs traffic to the home lab's gateway.     

*   **Internal Network Gateway**
    *   **pfSense Firewall** serves as the central security and routing appliance.
        *   **Responsibilities:** Intrusion Prevention/Detection (IPS/IDS), local DNS resolution, and hosting a **WireGuard VPN** server for secure remote access.
        *   **Connects to:** All internal hosts and provides a secure tunnel for remote devices.
*   **Internal Hosts & Services**
    
    *   **Synology NAS (Primary Docker Host):**
        
        *   **Hosts:** Core Docker Compose stack, including Media Managers (Radarr, Sonarr), Productivity Tools (Paperless-ngx), and other core applications.
            
    *   **Unraid Server (Secondary/Heavy-Workload Host):**
        
        *   **Hosts:** Services requiring significant storage or processing, such as large media file storage, download clients, and the centralized logging stack (ELK).
            
    *   **Mobile/Remote Devices:**
        
        *   **Connect:** Securely to the home lab via the WireGuard VPN on the pfSense firewall.
            
*   **Shared Security & Communication**
    
    *   **Inter-Host Communication:** The Synology NAS and Unraid Server communicate directly over the local network.
        
    *   **Unified Security Layer:** Both hosts are protected by a shared security stack, which includes:
        
        *   **CrowdSec:** For application-level intrusion detection.
            
        *   **Authentik:** For centralized identity and access management (IAM/SSO).
            
        *   **mTLS & Client Certificates:** For securing sensitive applications.

```

-------------------+---------------
|        External Network         |
+---------------------------------+
              |
              v
     [ Internet ]
              |
              v
+---------------------------------+
| Cloudflare (DNS, Proxy, CDN)    |
+---------------------------------+
              |
              v
+---------------------------------+
|     Internal Network (Home)     |
+---------------------------------+
              |
+-------------v-------------+
|      pfSense Firewall     |<------------------------------------------------------+
| (IPS/IDS, DNS, WireGuard) |                                                       |
+-------------+-------------+                                                       |
              |                                                                     |
              +----------------------------------+                                  |
              |                                  |                                  |
              v                                  v                                  v
+-------------+-------------+        +-------------+---------------+      +-------------+-------------+
|     Synology NAS (Host 1)   |      |      Unraid Server (Host 2) |      |   Mobile/Remote Devices   |
| (Docker Compose Primary)    |      |   (Heavy Workloads)     |   |      |  (WireGuard Clients)      |
+-----------------------------+      +-----------------------------+      +---------------------------+
| - Media Managers            |      | - Large Media Files         |
| - Productivity Tools        |      | - Downloads                 |
| - Core Apps                 |      | - Centralized Logging (ELK) |
+-----------------------------+      +-----------------------------+
              |                                  |
              |                                  |
              +-----------------+----------------+
                                |
                                v
                +-----------------------------+
                |    Shared Security Layer    |
                +-----------------------------+
                | - CrowdSec                  |
                | - Authentik (IAM)           |
                | - mTLS & Client Certs       |
                +-----------------------------+


```

This distributed architecture enhances both the performance and security posture of the entire self-hosted ecosystem.

Features
--------

*   **Automation:** Tools for automated media downloading and management.
*   **Media Serving:** Robust media server for movies, TV shows, and audiobooks.
*   **Financial Management:** Personal finance tracking and data import.
*   **Knowledge & Collaboration:** Wiki and bookmark management.
*   **System Monitoring:** Centralized logging, metrics collection, and visualization.
*   **Network Utilities:** DDNS updates, web proxy, and network analysis.
*   **Identity Management:** Authentication server for unified login across services.   
*   **Development Tools:** Browser-in-a-box solutions and a code server.
    

Prerequisites
-------------

Before you begin, ensure you have the following installed on your host machine:

*   **Docker:** [Installation Guide](https://docs.docker.com/get-docker/)
    
*   **Docker Compose:** Usually included with Docker Desktop, or available as a standalone installation: [Installation Guide](https://docs.docker.com/compose/install/)

Setup Instructions
------------------

### 1\. Clone the Repository

First, clone this repository to your local machine:

`   git clone https://github.com/upuldi/docker-compose.git  cd docker-compose   `

### 2\. Configure Environment Variables

**This is the most critical step for security.** The docker-compose.yaml file uses environment variables (e.g., $PUID, $PGID, $TZ, $DOCKER\_PATH, and various secrets). You must create a .env file in the root directory of this project (next to your docker-compose.yaml file) and populate it with your specific values.

**DO NOT COMMIT THIS .env FILE TO GIT.** Add .env to your .gitignore file.

Here's an example of what your .env file might look like. **Replace all placeholder values with your actual, secure information.**

``` plaintext
# User and Group IDs (find with 'id -u' and 'id -g' on your host)
PUID=1000
PGID=100

# Timezone (e.g., Europe/London, America/New_York, Australia/Sydney)
TZ=Australia/Sydney

# Base paths for Docker volumes
DOCKER_PATH=/path/to/your/docker/data
DOCKER_BACKUP_PATH=/path/to/your/docker-backups
MEDIA_PATH=/path/to/your/media
MEDIA_PATH_TOWER_SHARE=/path/to/your/media-tower
MEDIA_PATH_TV=/path/to/your/media/tv
MEDIA_PATH_MUSIC=/path/to/your/media/music
MEDIA_PATH_AUDIO=/path/to/your/media/audiobooks
MEDIA_PATH_BOOKS=/path/to/your/media/ebooks
PHOTO_PATH=/path/to/your/user/photos
NEXT_CLOUD_PATH=/path/to/your/user/nextcloud
ONLY_OFFICE_PATH=/path/to/your/user/onlyoffice
BABY_MEDIA_PATH=/path/to/your/baby_media # Example path, adjust as needed
YDL_PATH=/path/to/your/youtube_dl_data
DOCUMENT_PATH=/path/to/your/documents/

# Downloads
DOWNLOADS=/path/to/your/downloads/complete
INCOMPLETE=/path/to/your/downloads/incomplete

# Watchtower
WT_INTERVAL=21600 # Update interval in seconds (e.g., 21600 for 6 hours)
# WATCHTOWER_NOTIFICATION_URL="pushover://shoutrrr:YOUR_PUSHOVER_APP_API_SECRET@YOUR_PUSHOVER_USER_API_SECRET/?devices=YOUR_PUSHOVER_DEVICE telegram://YOUR_TELEGRAM_BOT_TOKEN@telegram?channels=YOUR_TELEGRAM_CHAT_ID pushbullet://YOUR_PUSHBULLET_API_SECRET"

# Authentik Specific
PG_USER=authentik
PG_PASS=your_authentik_db_password
AUTHENTIK_SECRET_KEY=your_authentik_secret_key
AUTHENTIK_ERROR_REPORTING__ENABLED=true
AUTHENTIK_EMAIL__HOST=smtp.example.com
AUTHENTIK_EMAIL__PORT=587
AUTHENTIK_EMAIL__USERNAME=your_email@example.com
AUTHENTIK_EMAIL__PASSWORD=your_email_password
AUTHENTIK_EMAIL__USE_TLS=true
AUTHENTIK_EMAIL__USE_SSL=false
AUTHENTIK_EMAIL__TIMEOUT=10
AUTHENTIK_EMAIL__FROM=your_email@example.com

# Plex Specific
HOST_NAME=your_plex_server_name
PLEX_CLAIM_TOKEN=your_plex_claim_token
PLEX_CLAIM_TOKEN_AUDIO=your_plex_audio_claim_token
ADVERTISE_IP= # e.g., http://192.168.1.100:32400 (leave blank if not applicable)
ADVERTISE_IP_AUDIO= # e.g., http://192.168.1.100:32400 (leave blank if not applicable)
ALLOWED_NETWORKS=10.27.28.0/24 # Comma-separated list of allowed networks/IPs

# Hoarder / Meilisearch
NEXTAUTH_SECRET=your_nextauth_secret
MEILI_MASTER_KEY=your_meili_master_key
NEXTAUTH_URL=http://10.27.29.49:3000 # Update with your Hoarder URL/IP

# Other Passwords/Tokens (Ensure these are set if corresponding services are enabled)
YOUR_LINKACE_DB_PASSWORD=
YOUR_MAIL_PASSWORD= # Generic mail password for various services
YOUR_FIREFLY_DB_PASSWORD=
YOUR_FIREFLY_APP_KEY=
YOUR_FIREFLY_ACCESS_TOKEN=
YOUR_DOCKER_USER= # For Docker Hub authentication (e.g., rest-api-planka)
YOUR_DOCKER_PASSWORD= # For Docker Hub authentication (e.g., rest-api-planka)
YOUR_WEBUI_SECRET_KEY= # For Deepseek/Ollama webui if enabled
YOUR_SPLUNK_PASSWORD=
YOUR_HEC_TOKEN=
YOUR_N8N_DB_PASSWORD=
YOUR_PGADMIN_PASSWORD=
YOUR_CODE_SERVER_PASSWORD=

# Authentik Image (Optional - if you want to pin a specific version)
# AUTHENTIK_IMAGE=ghcr.io/goauthentik/server
# AUTHENTIK_TAG=2024.6.3


```
### 3\. Deploy Services
Once your .env file is configured, you can deploy the services using Docker Compose:

    docker-compose up -d

This command will download the necessary Docker images, create the containers, and start them in the background.

To start specific services (e.g., only media-related ones if you've used profiles):

    docker-compose --profile download up -d

Service Breakdown
------------------
Below is a list of the services defined in the `docker-compose.yaml` file, their Docker images, and their primary purpose.

### Core Infrastructure

*   `autoheal`
    
    *   **Image:** `willfarrell/autoheal`
        
    *   **Purpose:** Automatically restarts unhealthy Docker containers based on their healthchecks.
        
*   `watchtower`
    
    *   **Image:** `containrrr/watchtower`
        
    *   **Purpose:** Monitors running Docker containers and automatically updates them to their latest image versions.
    
*   `dozzle`
    
    *   **Image:** `amir20/dozzle`
        
    *   **Purpose:** A real-time log viewer for Docker containers, accessible via a web interface.
        
*   `ddns-updater`
    
    *   **Image:** `qmcgaw/ddns-updater`
        
    *   **Purpose:** Dynamically updates DNS records for your domain with your current public IP address.
        
*   `postgres`
    
    *   **Image:** `postgres:15.1`
        
    *   **Purpose:** A robust relational database server, serving as the backend for many applications (Wiki.js, Firefly III, Planka, Authentik, n8n, etc.).
        
*   `redis`
    
    *   **Image:** `redis`
        
    *   **Purpose:** An in-memory data structure store, used as a cache and message broker for services like Paperless-ngx.
        
*   `tika`
    
    *   **Image:** `apache/tika`
        
    *   **Purpose:** Apache Tika is a content analysis toolkit that detects and extracts metadata and text from various document types, used by Paperless-ngx.
        
*   `gotenberg`
    
    *   **Image:** `docker.io/gotenberg:8.7`
        
    *   **Purpose:** A Docker-powered API for converting various document formats (HTML, Markdown, Office documents) into PDF files, used by Paperless-ngx.
        
*   `adminer`
    
    *   **Image:** `adminer`
        
    *   **Purpose:** A full-featured database management tool for managing various databases (PostgreSQL, MySQL, SQLite, etc.) via a web interface.

### Media Management

*   `prowlarr`
    
    *   **Image:** `linuxserver/prowlarr:nightly`
        
    *   **Purpose:** An indexer manager for your media libraries, integrating with your download clients and other media management tools.
        
*   `radarr`
    
    *   **Image:** `linuxserver/radarr:latest`
        
    *   **Purpose:** Movie collection manager for Usenet and BitTorrent users. It monitors RSS feeds for new movies and interacts with download clients.
        
*   `sonarr`
    
    *   **Image:** `lscr.io/linuxserver/sonarr:latest`
        
    *   **Purpose:** TV series management tool that monitors RSS feeds for new episodes and interacts with download clients.
        
*   `lidarr`
    
    *   **Image:** `linuxserver/lidarr:nightly`
        
    *   **Purpose:** Music collection manager for Usenet and BitTorrent users. It monitors for new music releases and interacts with download clients.
        
*   `readarr`
    
    *   **Image:** `lscr.io/linuxserver/readarr:develop`
        
    *   **Purpose:** eBook collection manager for Usenet and BitTorrent users, similar to Radarr and Sonarr but for books.
        
*   `qbittorrent`
    
    *   **Image:** `lscr.io/linuxserver/qbittorrent:latest`
        
    *   **Purpose:** A popular BitTorrent client with a web UI for managing downloads.
        
*   `jellyfin`
    
    *   **Image:** `lscr.io/linuxserver/jellyfin:latest`
        
    *   **Purpose:** A free software media system that lets you control your media from anywhere. It's an alternative to Plex or Emby.
        
*   `pinchflat`
    
    *   **Image:** `ghcr.io/kieraneglin/pinchflat:latest`
        
    *   **Purpose:** Appears to be a media downloading/processing tool, likely used for specific media types or automation.
        
*   `jackett`
    
    *   **Image:** `lscr.io/linuxserver/jackett:latest`
        
    *   **Purpose:** Acts as a proxy server for popular public and private torrent trackers, providing a single API for various indexers.
        
*   `plex`
    
    *   **Image:** `linuxserver/plex`
        
    *   **Purpose:** A comprehensive media server for organizing, playing, and streaming your movies, TV shows, music, and photos.
        
*   `plex\_audio`
    
    *   **Image:** linuxserver/plex
        
    *   **Purpose:** A dedicated Plex instance for organizing and streaming your audiobook collection.
    

### Financial & Productivity Tools

*   **firefly**
    
    *   **Image:** fireflyiii/core:latest
        
    *   **Purpose:** A self-hosted personal finance manager that helps you keep track of your expenses and income.
        
*   **fidi (Firefly III Data Importer)**
    
    *   **Image:** fireflyiii/data-importer:latest
        
    *   **Purpose:** A companion application for Firefly III that helps import financial data from various sources (e.g., CSV files).
        
*   **linkace**
    
    *   **Image:** linkace/linkace:simple
        
    *   **Purpose:** A self-hosted archive to collect, categorize, and permanently store your favorite links.
        
*   **paperless**
    
    *   **Image:** ghcr.io/paperless-ngx/paperless-ngx
        
    *   **Purpose:** A document management system that transforms your physical documents into a searchable online archive. It uses OCR for text recognition.
        
*   **vaultwarden**
    
    *   **Image:** vaultwarden/server:latest
        
    *   **Purpose:** An unofficial, lightweight Bitwarden server implementation written in Rust, providing a self-hosted password manager.
        
*   **huginn**
    
    *   **Image:** ghcr.io/huginn/huginn
        
    *   **Purpose:** A system for building agents that monitor and act on your behalf. It can create complex workflows and automations.
        
*   **rss-bridge**
    
    *   **Image:** rssbridge/rss-bridge
        
    *   **Purpose:** Generates RSS feeds for websites that don't have one, allowing you to follow content from almost any site.
        
*   **freshrss**
    
    *   **Image:** lscr.io/linuxserver/freshrss:latest
        
    *   **Purpose:** A free, self-hostable RSS feed reader that allows you to read news from all your favorite websites.
        
*   **hoarder**
    
    *   **Image:** ghcr.io/hoarder-app/hoarder:release
        
    *   **Purpose:** A digital archiving tool to save and organize web content.
        
*   **n8n**
    
    *   **Image:** docker.n8n.io/n8nio/n8n
        
    *   **Purpose:** A workflow automation tool that helps you integrate various services and automate tasks.
        
*   **planka**
    
    *   **Image:** ghcr.io/plankanban/planka:latest
        
    *   **Purpose:** A self-hosted project management tool, similar to Trello.
        
*   **rest-api-planka**
    
    *   **Image:** upuldi/planka-rest-api:latest
        
    *   **Purpose:** A custom REST API for the Planka project management tool, likely extending its functionality.
        

### Logging & Monitoring

*   **splunk**
    
    *   **Image:** splunk/splunk:8.2
        
    *   **Purpose:** A powerful platform for searching, monitoring, and analyzing machine-generated big data.
        
*   **loki**
    
    *   **Image:** grafana/loki:latest
        
    *   **Purpose:** A horizontally-scalable, highly-available, multi-tenant log aggregation system inspired by Prometheus. Designed to be cost-effective and easy to operate.
        
*   **promtail**
    
    *   **Image:** grafana/promtail:latest
        
    *   **Purpose:** An agent that ships logs from your local machines to a Loki instance. It's often run on each node to collect and forward logs.
        
*   **grafana**
    
    *   **Image:** grafana/grafana:latest
        
    *   **Purpose:** A leading open-source platform for monitoring and observability. It allows you to create dashboards and visualize data from various sources (like Loki and Splunk).
        
*   **fluent-bit**
    
    *   **Image:** cr.fluentbit.io/fluent/fluent-bit:latest
        
    *   **Purpose:** A fast and lightweight log processor and forwarder, often used to collect logs from various sources and send them to a centralized logging system.
        
*   **elasticsearch**
    
    *   **Image:** elasticsearch:7.9.1
        
    *   **Purpose:** A distributed, RESTful search and analytics engine, often used as the backend for logging solutions (ELK stack).
        
*   **kibana**
    
    *   **Image:** kibana:7.9.1
        
    *   **Purpose:** A data visualization and exploration tool used with Elasticsearch. It provides powerful dashboards and reports for analyzing data.
        

### Identity & Access Management

*   **authentic-postgresql**
    
    *   **Image:** docker.io/library/postgres:16-alpine
        
    *   **Purpose:** The PostgreSQL database backend for Authentik.
        
*   **authentic-redis**
    
    *   **Image:** docker.io/library/redis:alpine
        
    *   **Purpose:** The Redis cache for Authentik.
        
*   **authentic-server (authentik)**
    
    *   **Image:** ${AUTHENTIK\_IMAGE} (defaults to ghcr.io/goauthentik/server)
        
    *   **Purpose:** The main Authentik server responsible for identity and access management, including SSO, MFA, and user provisioning.
        
*   **authentic-worker**
    
    *   **Image:** ${AUTHENTIK\_IMAGE}
        
    *   **Purpose:** A worker process for Authentik, handling background tasks and integrations.
        

### Utilities & Development

*   **speedtest**
    
    *   **Image:** henrywhitaker3/speedtest-tracker
        
    *   **Purpose:** A self-hosted application for tracking your internet speed over time.
        
*   **chrome (for Hoarder)**
    
    *   **Image:** gcr.io/zenika-hub/alpine-chrome:123
        
    *   **Purpose:** A headless Chrome browser instance used by the hoarder service for web scraping and archiving.
        
*   **meilisearch**
    
    *   **Image:** getmeili/meilisearch:v1.11.1
        
    *   **Purpose:** A fast, open-source search engine used by the hoarder service for full-text search capabilities.
        
*   **obsidian**
    
    *   **Image:** lscr.io/linuxserver/obsidian:latest
        
    *   **Purpose:** A containerized version of Obsidian, a powerful knowledge base and note-taking application.
        
*   **code-server**
    
    *   **Image:** lscr.io/linuxserver/code-server
        
    *   **Purpose:** Runs VS Code in your browser, allowing you to access a full development environment from any device.

### Optional/Disabled Services

The docker-compose.yaml file includes several commented-out services that you can enable if needed. Their purposes are:

*   **portainer**: A Docker management UI for easily managing your containers, images, volumes, and networks.
    
*   **bazarr**: Companion application to Sonarr and Radarr that manages and downloads subtitles.
    
*   **posterr**: (Purpose unclear from name, likely related to media metadata or social posting.)
    
*   **gaps**: Finds missing movies in Plex libraries.
    
*   **mysql**: A relational database server, an alternative to PostgreSQL.
    
*   **firefox (jlesage/firefox)**: A desktop Firefox browser accessible via a web interface (VNC).
    
*   **neko (Chromium/Firefox)**: A tool for self-hosting a browser with shared control, useful for remote presentations or collaborative browsing.
    
*   **nextcloud-app**: A self-hosted file sync and share solution, like a personal Dropbox.
    
*   **onlyoffice-documentserver**: A powerful online office suite for collaborative document editing, often integrated with Nextcloud.
    
*   **onlyoffice-rabbitmq**: A message broker used by OnlyOffice.
    
*   **ytdl\_material**: A web UI for youtube-dl (or yt-dlp), allowing you to download videos from various platforms.
    
*   **ytdl-mongo-db**: The MongoDB backend for ytdl\_material.
    
*   **gitlab**: A complete DevOps platform, including Git repository management, CI/CD, and more.
    
*   **jenkins**: An open-source automation server for building, testing, and deploying software.
    
*   **monica**: A personal relationship management (PRM) system to keep track of your interactions with friends and family.
    
*   **babybuddy**: A web application to help parents track their baby's activities (feedings, diaper changes, sleep, etc.).
    
*   **photoprism**: An AI-powered app for browsing, organizing, and sharing your photo collection.
    
*   **webui (for Deepseek/Ollama)**: A web user interface for interacting with large language models (LLMs) served by Ollama.
    
*   **ollama**: Runs large language models locally on your machine.
    

To enable any of these, simply uncomment their respective sections in the docker-compose.yaml file and ensure all necessary environment variables are set in your .env file.

Network Configuration
---------------------

The docker-compose.yaml defines two custom networks:

*   **dockervlan (MacVLAN)**: This network is configured as a macvlan, which allows containers to have their own unique MAC addresses and IP addresses on your physical network. This means they appear as separate devices on your LAN, making them directly accessible from other devices on your network without port forwarding on the host.
    
    *   **parent**: **Critical:** You _must_ update ovs\_eth0 to match the name of your host machine's physical network interface (e.g., eth0, enp0s3).
        
    *   **subnet**: The IP range for this network. Ensure it's within your LAN's subnet but outside your router's DHCP range to avoid conflicts.
        
    *   **gateway**: Your router's IP address.
        
    *   **aux\_addresses**: host is an optional IP assigned to the host within the macvlan network, useful for host-to-container communication.
        
*   **host\_network (Bridge)**: This is a standard Docker bridge network. Containers connected to this network can communicate with each other and the host, but are not directly exposed to the external network without explicit port mapping.

Usage and Management
--------------------

*  **Start all services:**  ```bash docker-compose up -d  ```
*  **Start specific services (e.g., download services):** ```bash docker-compose --profile download up -d   ```   
*  **Stop all services:** ```bash docker-compose down      ```
*  **Stop and remove containers, networks, and volumes (data volumes are preserved by default):** ```bash docker-compose down -v # Use -v to remove anonymous volumes too      ```
*   **View logs for all services:** ```bash docker-compose logs -f      ```
*   **View logs for a specific service (e.g., `plex`):** ```bash docker-compose logs -f plex      ```
*   **Restart a service (e.g., `radarr`):** ```bash docker-compose restart radarr     ```
*   **Update all images and recreate containers (pulls latest image and rebuilds):** ```bash docker-compose pull && docker-compose up -d --remove-orphans   ```


Troubleshooting
---------------

*   **Container not starting:**
    
    *   Check docker-compose logs for error messages.
        
    *   Verify your .env file for correct sensitive values and paths.
        
    *   Ensure port conflicts are not occurring (check ports section in docker-compose.yaml and host netstat -tuln).
        
*   **macvlan issues:**
    
    *   Double-check the parent interface name in the dockervlan network configuration. It must exactly match an active network interface on your host.
        
    *   Ensure the subnet and gateway are correctly configured for your home network. The assigned ipv4\_address for containers must be outside your router's DHCP range but within the defined subnet.
        
    *   Verify your firewall rules on the host; macvlan traffic bypasses the host's networking stack directly.
        
*   **Permissions issues:**
    
    *   Ensure the PUID and PGID in your .env file match a user and group on your host that has appropriate read/write permissions to the mounted volumes.
        
    *   Check volume mount paths in docker-compose.yaml to ensure they exist and are accessible from the Docker daemon.
        

Customization
-------------

*   **Enable/Disable Services:** Comment out or uncomment service blocks as desired.
    
*   **Adjust Resource Limits:** For services like bazarr that have mem\_limit and mem\_reservation, you can adjust these to control resource usage on your host.
    
*   **Modify Ports:** Change the ports mappings (e.g., HOST\_PORT:CONTAINER\_PORT) to suit your needs.
    
*   **Update Images:** Change image:tag to pin specific versions or use latest (with caution for production).
    
*   **Add New Services:** Integrate additional Docker services by defining them in the services section.