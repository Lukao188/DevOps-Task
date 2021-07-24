# Implementation

## Pre-Requisites:
- Install Docker Desktop following instructions from: https://docs.docker.com/docker-for-windows/install/
- Install Git Bash from: https://git-scm.com/downloads
- Optional: Set VSCode as the default text editor, copy the contents from MyGitConfigSettings into the config file
```
git config --global core.editor code
git config --global -e
```
## Task Steps
- Note: It is possible to start the entire project in the folder where the compose file is located using only `docker compose up -d` but instead we will focus on building each container one by one.
- Note: It is possible to remove all stopped containers, dangling images, and unused networks using `docker system prune`
### Part I) Jenkins initial setup:
- Go into the folder where the compose file is located i.e. `cd Files`.
- Create the following directory `mkdir jenkins_data` This volume will be used to persist all your jenkins data: configurations, plugins, pipelines, passwords, etc.
- Pull the latest Jenkins LTS image from Docker Hub `docker pull jenkins/jenkins:lts`
- Start the jenkins container (Also used to recreate containers) `docker-compose up -d jenkins` 
  in detached -d mode so that the Docker container runs in the background of your terminal 
- Get the initial admin password `docker exec jenkins cat var/jenkins_home/secrets/initialAdminPassword`
- Confirm the jenkins container is running `docker ps`
- Navigate to the Jenkins URL in your browser: http://localhost:8080 using the initial password to access and setup Jenkins selecting install suggested plugins. After that create your admin user. 
- Since we are using a custom Docker image for the Jenkins container, specified in the Dockerfile, there is no need for manual installation of additional plugins.
- However, in order to confirm this, go to manage Jenkins/manage Plugins/installed and check whether the following plugins been installed:
1. Nexus Artifact Uploader
2. Pipeline Utility Steps
3. Gogs
4. Pipeline Maven Integration
5. Maven Integration
6. Config File Provider
- If any are missing install them manually or update them to the latest version available.
- After the download is finished tick restart jenkins and wait for it to reboot
- Set up maven as a managed tool Configuration/Global Tool Configuration/Maven installations name it jenkins-maven and select the latest version.

### Part II) Nexus initial setup:
- Go into the folder where the compose file is located i.e. `cd Files`.
- Create the following directory `mkdir nexus` This volume will be used to persist all your nexus data: configurations, plugins, pipelines, passwords, etc.
- Pull the Sonatype Nexus 3 image from Docker Hub `docker pull sonatype/nexus3`
- Start the nexus container (Also used to recreate containers) `docker-compose up -d nexus`
- Get the initial password for admin user `docker exec nexus3 cat nexus-data/admin.password`
- Test out if Jenkins can properly talk to Nexus using by typing in our terminal `winpty docker exec -it jenkins bash` and then inside the container `curl -v nexus3:8081` -v means in verbose mode so we can get a detailed response.
- Navigate to the Nexus artifact repository URL http://localhost:8081 using your browser to access Nexus and log in with admin user and the initial password. Change the password and enable anonymous access.
- Go into settings and create a new local user called jenkins-user. Grant it nx-admin rights. Log in with this user.
- In settings/repositories create a hosted repository where we will upload our artifacts. Create/maven2(hosted), name it maven-nexus-repo, version policy mixed and allow redeploy.
- In Jenkins under Config/Managed Files/Add a new config/Global Maven Settings.xml, give it name and ID values: jenkins-nexus-config and copy the .xml contents into it.
- Make sure to enter your Nexus Credentials in the username/passwrod fields.

### Part III) Gogs initial setup:
- Go into the folder where the compose file is located i.e. `cd Files`.
- Create the following directory `mkdir -p gogs/data` This volume will be used to persist all your gogs data: configurations, plugins, pipelines, passwords, etc.
- Pull the Gogs image from Docker Hub `docker pull gogs/gogs`
- In the docker-compose file Gogs container is configured so that it serves Git’s Web interface at localhost:10080, and ssh at localhost:10022 for external communication.
- Start the Gogs container (Also used to recreate containers) `docker-compose up -d gogs`
- Test out if Gogs can properly talk to Jenkins by typing in our terminal `winpty docker exec -it gogs bash` and then inside the container `curl -v jenkins:8080` -v means in verbose mode so we can get a detailed response.
- Navigate to the Gogs repository URL http://localhost:10080 using your browser and complete the initial setup, select SQLite3 and leave other options as default. After the installation you need to manually navigate to http://localhost:10080/user/login since we mapped container's port 3000 to our external host port 10080. Sing up with a user: git-user after which you need ti create a new repository: first-repo
- If you don't have one create an RSA key pair in your home directory `ssh-keygen -t rsa` go into the created directory `cd .ssh/` and copy the contents of id_rsa.pub key to the Gogs repository settings/ssh keys/add key, name it first-pub-key.
- To test the SSH connectivity run `ssh -T git@localhost -p 10022`

### Additional info:
- In order to see the logs of the container `docker-compose logs -f -t <container_name>`
- list all docker images `docker images -a`
- List all docker containers `docker ps -a`
- Stop the container `docker stop <container_id>`
- Remove the container `docker rm <container_id>`
- Remove the image `docker rmi <image_id>`
- Remove the volume that is attached to the container `docker-compose rm -v <container_name>`

### Part IV) Configuring the jenkins pipeline job:
- Create a new freestyle project on jenkins to test the connectivity with Gogs and Nexus. In the build step choose execute shell, enter the following
`curl -v nexus3:8081 && curl -v gogs:3000` and build the job.
- Clone the needed repo `git clone https://github.com/stevancvetkovic/java-app-sample.git`.
- Make a local repo in order to push the java app sample to our Gogs instance:
```
mkdir first-repo 
cd first-repo/ 
git init
cp -r ../java-app-sample/* .
git add .
git commit -m "Added java app to the repo"
git remote add origin ssh://git@localhost:10022/git-user/first-repo.git
git push -u origin master
```
- On jenkins under settings/manage credentials/jenkins/global credentials/ add your nexus and gogs credentials.
- ID: first-repo username/password: git-user/***
- ID: nexus-user-credentials username/password: jenkins-user/***
- Create a Jenkins Pipeline job (Groovy), name it Devops_Task, copy the Jenkinsfile contents into the pipeline and save the build.
- On Gogs add a new webhook in http://localhost:10080/git-user/first-repo/settings/hooks/ of type Gogs, with Payload URL: http://jenkins:8080/gogs-webhook/?job=Devops_Task a Secret you want and select just the push event.
- Back in the Jenkins job under Gogs Webhook, tick Use Gogs Secret and enter the secret you chose.
- You can now test the build by pushing any change to your Gogs repo which will trigger a build and send the artifacts to Nexus.
- Under Nexus Browse/maven-nexus-repo we can see our build artifacts. Expand the specific build, select the file that you want to inspect and on the right-hand side click on the path which will download the file.
