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

    # Docker Network Configuration (adjust parent and subnet for your network)
    # For macvlan:
    # MACVLAN_PARENT_INTERFACE=ovs_eth0 # Example: replace with your host's network interface (e.g., eth0, enpXs0)
    # MACVLAN_SUBNET="10.27.28.0/20"
    # MACVLAN_GATEWAY="10.27.28.1"
    # MACVLAN_AUX_ADDRESS_HOST="10.27.28.166"
    # For host_network (bridge):
    # BRIDGE_SUBNET="172.29.0.1/24"
    # BRIDGE_IP_RANGE="172.29.0.1/24"
    # BRIDGE_GATEWAY="172.29.0.1"
```
### 3\. Deploy Services
Once your .env file is configured, you can deploy the services using Docker Compose:

    docker-compose up -d

This command will download the necessary Docker images, create the containers, and start them in the background.

To start specific services (e.g., only media-related ones if you've used profiles):

    docker-compose --profile download up -d

Service Breakdown
------------------
Below is a list of the services defined in the docker-compose.yaml file, their Docker images, and their primary purpose.