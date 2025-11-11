# MERN Stack CI/CD Pipeline

This repository contains a Jenkins pipeline for automating the **build, tag, push, and cleanup** process of MERN stack Docker images (Server and Client) to Docker Hub.

---

## Table of Contents

- [Prerequisites](#prerequisites)  
- [Pipeline Overview](#pipeline-overview)  
- [Jenkins Pipeline Stages](#jenkins-pipeline-stages)  
- [Docker Hub Repository](#docker-hub-repository)  
- [Cleanup](#cleanup)  
- [Post Actions](#post-actions)  
- [Usage](#usage)  

---

## Prerequisites

1. **Jenkins Server**  
   Ensure Jenkins is installed and configured with the necessary plugins:
   - Pipeline
   - Docker Pipeline
   - Credentials Binding  

2. **Docker Hub Account**  
   Make sure you have a Docker Hub account and credentials stored in Jenkins as a secret:
   - Username: `dockerhub`
   - Password: `dockerhub`

3. **Repository Setup**  
   The repository must contain:
   - `docker-compose.yml` defining the `server` and `client` services.

4. **SCM Access**  
   Jenkins must have access to the Git repository containing the MERN project.

---

## Pipeline Overview

This pipeline performs the following tasks automatically whenever a change is detected in the repository:

1. Checkout code from Git.
2. Login to Docker Hub.
3. Build Docker images for the Server and Client.
4. Tag images with build number and `latest`.
5. Push images to Docker Hub.
6. Cleanup old Docker images.
7. Logout from Docker Hub.

The pipeline triggers every **5 minutes** to poll the SCM for changes.

---

## Jenkins Pipeline Stages

### 1. Checkout

- Pulls the latest code from the Git repository.
- Confirms checkout completion.

### 2. Docker Login

- Authenticates with Docker Hub using credentials stored in Jenkins.
- Ensures subsequent Docker commands work properly.

### 3. Build Docker Images

- Builds Docker images for the MERN **Server** and **Client** services.
- Uses the current build number as the image tag.
- Confirms built images using `docker images`.

### 4. Tag Images as Latest

- Tags both Server and Client images with the `latest` tag.
- Confirms tagged images.

### 5. Push to Docker Hub

- Pushes both **build-number-tagged** and **latest** images to Docker Hub.
- Provides links to the repositories.

### 6. Cleanup Old Images

- Keeps the latest and current build images only.
- Removes old images to save space.
- Removes dangling images using `docker image prune -f`.

### 7. Docker Logout

- Logs out from Docker Hub to secure credentials.

---

## Docker Hub Repository

- Server Image: `dali917/mern-server`  
- Client Image: `dali917/mern-client`  

Access pushed images at:  
- [Server](https://hub.docker.com/r/dali917/mern-server/tags)  
- [Client](https://hub.docker.com/r/dali917/mern-client/tags)  

---

## Cleanup

- Old Docker images (except `latest` and current build) are removed automatically.
- Dangling images are pruned to save disk space.

---

## Post Actions

- On **success**, the pipeline prints a summary including build number and Docker Hub repositories.
- On **failure**, the pipeline prints an error summary.
- Always performs cleanup at the end, including logging out from Docker Hub and removing unused Docker resources.

---

## Usage

1. Add this `Jenkinsfile` to your repository.
2. Configure Jenkins job:
   - Pipeline type: Pipeline script from SCM.
   - Branch: `master`
   - Credentials: Docker Hub credentials.
3. Trigger the job manually or let it run on SCM polling.
4. Verify the images in Docker Hub.

---

This setup provides **fully automated CI/CD** for MERN stack projects using Jenkins and Docker Hub.
