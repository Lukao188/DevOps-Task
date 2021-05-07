# Implementation Steps

## Pre-Requisites:
- This document is taking into consideration that this will be run on the Windows OS.
- Install Docker Desktop following instructions from: https://docs.docker.com/docker-for-windows/install/
- Install Git Bash from: https://git-scm.com/downloads
- Clone this repository via `git clone https://github.com/Lukao188/DevOps-Task.git` or Download the files into a folder.
- Go into the folder where the files are located i.e. `cd Files` and `rm -rf .git/` if you cloned the repo.

## Installation:
- Note: It is possible to start the entire project using only `docker compose up -d` but in the next part we will cover building each container one by one.
- Note: It is possible to remove all stopped containers, dangling images, and unused networks using `docker system prune`
### Part I) Jenkins initial setup:
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
- Remove the volume that is attached to the container `docker-compose rm -v jenkins`

### Part II) Nexus initial setup:
- Pull the Sonatype Nexus 3 image from Docker Hub `docker pull sonatype/nexus3`
- Start the nexus container (Also used to recreate containers) `docker-compose up -d nexus`
- Get the initial password for admin user `docker exec nexus3 cat nexus-data/admin.password`
- Test out if Jenkins can properly talk to Nexus using by typing in our terminal `winpty docker exec -it jenkins bash` and then inside the container `curl -v nexus3:8081` -v means in verbose mode so we can get a detailed response.
- Navigate to the Nexus artifact repository URL http://localhost:8081 using your browser to access Nexus and log in with admin user and the initial password. Change the password and enable anonymous access.
- Go into settings and create a new local user called jenkins-user. Grant it nx-admin rights. Log in with this user.
- In settings/repositories create a hosted repository where we will upload our artifacts. Create/maven2(hosted), name it maven-nexus-repo, version policy mixed and allow redeploy.

- In order to see the logs of the container `docker-compose logs -f -t nexus`
- Stop the container `docker compose stop nexus`
- Remove the container `docker compose rm -f nexus`
- Remove the volume that is attached to the container `docker-compose rm -v nexus`

### Part III) Gogs initial setup:
- Pull the Gogs image from Docker Hub `docker pull gogs/gogs`
- In the docker-compose file Gogs container is configured so that it serves Git’s Web interface at localhost:10080, and ssh at localhost:10022 for external communication.
- Start the Gogs container (Also used to recreate containers) `docker-compose up -d gogs`
- Test out if Gogs can properly talk to Jenkins by typing in our terminal `winpty docker exec -it gogs bash` and then inside the container `curl -v jenkins:8080` -v means in verbose mode so we can get a detailed response.
- Navigate to the Gogs repository URL http://localhost:10080 using your browser and complete the initial setup, select SQLite3 and leave other options as default. After the installation you need to manually navigate to http://localhost:10080/user/login since we mapped container's port 3000 to our external host port 10080. Sing up with a user: git-user after which you need ti create a new repository: first-repo
- If you don't have one create an RSA key pair in your home directory `ssh-keygen -t rsa` go into the created directory `cd .ssh/` and copy the contents of id_rsa.pub key to the Gogs repository settings/ssh keys/add key, name it first-pub-key.
- To test the SSH connectivity run `ssh -T git@localhost -p 10022`

- In order to see the logs of the container `docker-compose logs -f -t gogs`
- Stop the container `docker compose stop gogs`
- Remove the container `docker compose rm -f gogs`
- Remove the volume that is attached to the container `docker-compose rm -v gogs`

### Part IV) Configuring the pipeline:
- Create a freestyle job on jenkins to test the connectivity with Gogs and Nexus. In the build step choose execute shell and enter:
``` curl -v nexus3:8081
curl -v gogs:3000
```
- Clone the needed repo `git clone https://github.com/stevancvetkovic/java-app-sample.git`.
- Make a local repo in order to push the java app sample to our Gogs instance:
```mkdir first-repo 
cd first-repo/ 
git init
cp -r ../java-app-sample/* .
git add .
git commit -m "Added java app to the repo"
git remote add origin ssh://git@localhost:10022/git-user/first-repo.git
git push -u origin master
```
