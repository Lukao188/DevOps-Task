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
  in detached -d mode so that the Docker container runs in the background of your terminal 
- Get the initial admin password `docker exec jenkins cat var/jenkins_home/secrets/initialAdminPassword`
- Confirm the jenkins container is running `docker ps`
- Navigate to the Jenkins URL in your browser: http://localhost:8080 using the initial password to access and setup Jenkins selecting install suggested plugins. After that create your admin user. 
- Go to manage Jenkins/manage Plugins/available, select the following plugins and select install without restart:
1. Nexus Artifact Uploader
2. Pipeline Utility Steps
3. Gogs
4. Pipeline Maven Integration
5. Maven Integration
- After the download is finished tick restart jenkins and wait for it to reboot

- In order to see the logs of the container `docker-compose logs -f -t jenkins`
- Stop the container `docker compose stop jenkins`
- Remove the container `docker compose rm -f jenkins`
- Remove the volumes that are attached to the container `docker-compose rm -v jenkins`

### Nexus initial setup:
- Pull the Sonatype Nexus 3 image from Docker Hub `docker pull sonatype/nexus3`
- Start the nexus container (Also used to recreate containers) `docker-compose up -d nexus3`
- Get the initial password for admin user `docker exec nexus3 cat nexus-data/admin.password`
- Test out if Jenkins can properly talk to Nexus using by typing in our terminal `winpty docker exec -it jenkins bash` and then inside the container `curl -v nexus3:8081` -v means in verbose mode so we can get a detailed response.
- Navigate to the Nexus repository URL http://localhost:8081 using your browser to access Nexus and log in.
