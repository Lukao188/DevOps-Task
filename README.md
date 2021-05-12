# DevOps Task

## Software in use:
-	Docker Desktop
-	Jenkins
-	Nexus
-	Gogs

## Part I
1.	Create Docker Compose file used to launch Jenkins (latest LTS version) using official Jenkins Docker image and persistent Docker volume for Jenkins Home directory
2.	Create a README file next to the Compose file to document how to create and terminate Jenkins container using Compose commands

## Part II
1.	Expand Docker Compose file to launch Nexus v3.30.1 using official Docker image from https://hub.docker.com/r/sonatype/nexus3
2.	Ensure Nexus container is using persistent storage for Nexus artifacts with Docker volumes
3.	Ensure Jenkins jobs can talk to Nexus, test it out using curl/wget. If they don’t, link containers using Compose file to establish network connection between them.
4.	Update a README file next to the Compose file to document how to create and terminate Nexus container using Compose commands

## Part III
1.	Expand Docker Compose file to launch Gogs version 0.12.3
2.	Ensure Gogs container is using persistent storage for Gogs repositories with Docker volumes
3.	Ensure Jenkins jobs can talk to Gogs, test it out with curl/wget, but also with a temporary Jenkins job which clones a repository from Gogs
4.	Ensure Gogs can talk to Jenkins – from command line after entering into shell of the Gogs container and then using curl/wget, or through Gogs repository Webhook that hits a Jenkins job. Take necessary actions in Compose file to establish network connectivity.
5.	Update a README file next to the Compose file to document how to create and terminate Jenkins container using Compose commands

## Part IV
1.	Grab a sample Java app from https://github.com/stevancvetkovic/java-app-sample and push it to your local Gogs instance
2.	Create a Jenkins Pipeline job (Groovy) that clones local Gogs repo with this app and runs a build using Maven command “mvn clean deploy”. Maven “deploy” command compiles the app and pushes it to configured repository – make sure the package gets delivered to your local Nexus.
3.	Configure this Jenkins job to run automatically on every push to “master” branch in Gogs of the app repository. The job will still compile and package the app and push it to Nexus. Bonus points: find a way with Maven how to have unique app version with each build, so we don’t overwrite an existing package in Nexus, but have each build produce a separate one.
4.	Update a README file next to the Compose file to include info on how application build process is executed and where the compiled package is stored and how to retrieve it for manual inspection.


## Deliverables:
-	A single Docker Compose file
-	A single Jenkinsfile for sample Java app
-	A single README file
-	Any other files you may find necessary for this setup to work.
