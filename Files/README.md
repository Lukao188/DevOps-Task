# Implementation Steps

## Pre-Requisites:
- Taking into consideration that this will be run on Windows OS.
- Install Docker Desktop following instructions from: https://docs.docker.com/docker-for-windows/install/
- Install Git Bash from: https://git-scm.com/downloads
- Clone this repository via `git clone https://github.com/Lukao188/DevOps-Task.git` or Download the files into a folder.
- Go into the folder where the files are located i.e. `cd Files`

## Installation:
- Note: It is possible to start the entire project using only `docker compose up -d` but in the next part we will cover building each container one by one.
- Note: It is possible to remove all stopped containers, dangling images, and unused networks using `docker system prune`
### Jenkins initial setup:
- Pull the latest Jenkins LTS image from Docker Hub `docker pull jenkins/jenkins:lts`
- Start the jenkins container (Also used to recreate containers) `docker-compose up -d jenkins`
- Get the initial admin password `docker exec jenkins cat var/jenkins_home/secrets/initialAdminPassword`
- Confirm the jenkins container is running `docker ps`
- Navigate to the URL in your browser: http://localhost:8080 using the initial password to access and setup Jenkins selecting install suggested plugins.
- Stop the container `docker compose stop jenkins`
- Remove the container `docker compose rm -f jenkins`
- Remove the volumes that are attached to the jenkins container `docker-compose rm -v jenkins`
