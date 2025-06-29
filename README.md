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

## Home Lab Architecture

This Docker Compose setup is designed to integrate into a robust home lab environment, distributed across multiple hosts for optimal performance, security, and data management. All self-hosted services are accessed via HTTPS, secured with certificates from Let's Encrypt. Cloudflare is utilized as the external DNS provider, pointing domains to the public IP and leveraging its features.