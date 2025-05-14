Overview

Jenkins is a leading open-source automation server used to build, test, and deploy software. This guide focuses solely on setting up Jenkins itself, running it in Docker, and configuring basic jobs.

Prerequisites

Docker (v20.10+)

Docker Compose (optional)

Access to a terminal with Docker privileges

Ensure you can run Docker commands:

``` docker version ```
``` docker-compose version ```

#Quickstart: Jenkins in Docker

1. Pull the Jenkins LTS image

``` docker pull jenkins/jenkins:lts ```

2. Run Jenkins container

```` docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts ````

  
Ports:

8080: Jenkins UI

50000: Agent connections

Volume: jenkins_home persists all Jenkins data


Production Setup: Docker Compose

**Create a docker-compose.yml:**
````
version: '3.8'
services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    restart: unless-stopped
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
volumes:
  jenkins_home:

  ````

#Accessing Jenkins

Open your browser at http://localhost:8080

Retrieve the initial admin password:

docker logs jenkins | grep 'initialAdminPassword'

Unlock Jenkins by entering the password in the UI

Install suggested plugins




