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

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   git clone https://github.com/your-username/your-repo-name.git  cd your-repo-name   `

### 2\. Configure Environment Variables

**This is the most critical step for security.** The docker-compose.yaml file uses environment variables (e.g., $PUID, $PGID, $TZ, $DOCKER\_PATH, and various secrets). You must create a .env file in the root directory of this project (next to your docker-compose.yaml file) and populate it with your specific values.

**DO NOT COMMIT THIS .env FILE TO GIT.** Add .env to your .gitignore file.

Here's an example of what your .env file might look like. **Replace all placeholder values with your actual, secure information.**