[![Build Status](https://travis-ci.com/Otus-DevOps-2019-08/SergeyKa-cmd_microservices.svg?branch=master)](https://travis-ci.com/Otus-DevOps-2019-08/SergeyKa-cmd_microservices)
# SergeyKa-cmd_microservices
### Contents:
  ## 1. Docker: First look
  ## 2. Docker: Containers & Images maintain
  ## 3. Docker: Images & Microservices
  ## 4. Docker: Networking & Docker-compose implementation
  ## 5. Gitlab: Deployment & pipeline preparations
  ## 6. Monitoring: Prometheus configuring and deployment
_______________________________________________________________________________________________________
## 1. Docker: First look
### Main issue: docker host & image creation, docker hub registry
### Additional task: docker container and docker image files comparison
## System prerequisites
  + Prepare for [Docker environment](https://docs.docker.com/install/linux/docker-ce/ubuntu/);
  + Prepare for [Docker Compose](https://docs.docker.com/compose/install/);
## App testing for additional task:
  + Comparing two files: inspect_container.json and inspect_image.json using commands:
    
    $docker inspect <CONTAINER ID> > inspect_container.json
    
    $docker inspect <IMAGE ID> > inspect_image.json
  
    $diff inspect_image.json inspect_container.json
  + Save all comparison information and docker images result to file docker-1.log:
  
    $docker images > docker-1.log
_____________________________________________________________________________________________________________________________
## 2. Docker: Containers & Images maintain
### Main issue: Docker integration to GCE and Docker Hub
### Additional task: Up and running instances in GCE using Packer & Terraform & Ansible with docker image from Docker Hub
## System prerequisites:
  + Create new project in [GCE](https://console.cloud.google.com/compute)
  + Prepare for [GCloud SDK](https://cloud.google.com/sdk/) on local machine
  + Register on [Docker Hub](https://hub.docker.com/)
 ## App testing for additional task:
  + Clone current preository and prepare own packer/variables.json and terraform/stage/variables.tf files
  + Check that app is up and running via docker run:
  
    $docker run --name reddit -d -p 9292:9292 sergeykacmd/otus-reddit:1.0
  + Checkout current inventory in [Google Cloud Console](https://console.cloud.google.com/compute)
  + Run Packer & Terraform & Ansible commands:
  
    $packer buid -var-file=variables.json app.json && packer buid -var-file=variables.json db.json
    
    $terraform apply
    
    $ansible-playbook playbooks/site.yml
_________________________________________________________________________________________________
## 3. Docker: Images & Microservices
### Main issue: Decomposition of application for microservice environment
### Additional task: Tuning current dockerfiles: changing network aliases; using Alpine Linux packages for post, comment and ui.
## System prerequisites:
  + Prepare for [GCloud SDK](https://cloud.google.com/sdk/) on local machine
  + Register on [Docker Hub](https://hub.docker.com/)
 ## App testing:
  + Clone current repository to your environment
  + Ensure that your docker host up and running via terminal:

    $ docker-machine ls
  + Create docker images using terminal:
    
    $ docker build -t your-dockerhub-login/post:1.0 ./post-py
    
    $ docker build -t your-dockerhub-login/comment:1.0 ./comment
    
    $ docker build -t your-dockerhub-login/ui:1.0 ./ui
  + Create your private network:
    
    $ docker network create
  + Finally run containers for app:
    
    $ docker run -d --network=your-network-tag --network-alias=post_db --network-alias=comment_db mongo:latest
    
    $ docker run -d --network=your-network-tag --network-alias=post your-dockerhub-login/post:1.0
    
    $ docker run -d --network=your-network-tag --network-alias=comment your-dockerhub-login/comment:1.0
    
    $ docker run -d --network=your-network-tag -p 9292:9292 your-dockerhub-login/ui:1.0
  
``` docker-machine ls                                          
     NAME          ACTIVE   DRIVER   STATE     URL                       SWARM   DOCKER     ERRORS
     docker-host   -        google   Running   tcp://35.195.87.92:2376           v19.03.4 

  docker ps
    CONTAINER ID        IMAGE                     COMMAND                  CREATED              STATUS              PORTS               NAMES
    6e358cd62854        sergeykacmd/post:1.1      "python3 post_app.py"    About a minute ago   Up About a minute                       busy_kalam
    e1b43ad457de        sergeykacmd/comment:1.1   "puma"                   2 minutes ago        Up About a minute                       infallible_swanson
    3ace7552e21b        mongo:latest              "docker-entrypoint.s…"   2 minutes ago        Up 2 minutes        27017/tcp           confident_lamarr

docker images                       
    REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
    sergeykacmd/ui        1.1                 e67770bf33aa        4 minutes ago       281MB
    sergeykacmd/post      1.1                 57c520178183        5 minutes ago       109MB
    sergeykacmd/comment   1.1                 19a9095fce88        6 minutes ago       278MB
    mongo                 latest              965553e202a4        2 weeks ago         363MB
    alpine                3.10                965ea09ff2eb        3 weeks ago         5.55MB
    python                3.6.0-alpine        cb178ebbf0f2        2 years ago         88.6MB
```
 ## App connectivity testing:
  + Ensure that your Monolith Reddit up and running http://:9292
  + [Monolith Reddit](http://35.195.87.92:9292/) - to test my solution
__________________________________________________________________________________
 # 4. Docker: Networking & Docker-compose implementation
 ### Main issue: Network features discovering in docker, docker-compose implementation 
 ### Additional task: Docker-compose-override file creation
 ## System prerequisites:
  + Connect to current running GCE instance using command:
  
    $ eval $(docker-machine env docker-host)
 ## App testing:
  + Clone current repository to your environment
  + Ensure that your docker host up and running via terminal:

    $ docker-machine ls
  + Create docker container using network "none-driver":
    
    $ docker run -ti --rm --network none joffotron/docker-net-tools -c ifconfig
  + Create docker container using network "host-driver":
    
    $ docker run -ti --rm --network host joffotron/docker-net-tools -c ifconfig
  + Create docker container using network "bridge-driver":

    $ docker network create reddit --driver bridge
    
    $ docker network create back_net --subnet=10.0.2.0/24
    
    $ docker network create front_net --subnet=10.0.1.0/24
   + Connect containers to user-defined "front_net" network
    
    $ docker network connect front_net post
    
    $ docker network connect front_net comment
  + Ensure that [Monolith Reddit](http://35.195.87.92:9292/) up and running
  + Prepared for .env file with user-defined variables in docker-compose.yml file
  + Prepared docker-compose.override.yml file that aoutomatically roll-out within docker-compose.yml file:
  
    $ docker-compose up -d
  + Ensure that all containers are implemented by using docker-compose ps:
  
  ```docker-compose ps
  Name                  Command             State           Ports         
----------------------------------------------------------------------------
src_comment_1   puma --debug -w 2             Up                            
src_post_1      python3 post_app.py           Up                            
src_post_db_1   docker-entrypoint.sh mongod   Up      27017/tcp             
src_ui_1        puma --debug -w 2             Up      0.0.0.0:9292->9292/tcp
 ```
  + All docker-compose entity have project related pefix (at this point is "src_") which is the name of current project directory.
_______________________________________________________________________________________________
  ## 5. Gitlab: Deployment & pipeline preparations
  ### Main issue: Docker-based Gitlab deployment on GCP instance, pipeline implementation
  ### Additional task: Design and implement scalable solution for Gitlab Runner in one configuration file & Chat Ops implement
  ## System prerequisites:
  + Prepare new instance using [gist](https://gist.githubusercontent.com/SergeyKa-cmd/b761cb0f4c1c9cb363600a177eebfb26/raw/7490234f952cad68a0df1756013ace6efb56f103/gistfile1.txt) from scrach and run it from terminal
  + Connect to current running GCE instance using command:
  
    $ eval $(docker-machine env docker-host)
    
    $ docker-machine ssh docker-host
  + Prepare docker environment on instance using snippet:
  ```# mkdir -p /srv/gitlab/config /srv/gitlab/data /srv/gitlab/logs
     # cd /srv/gitlab/
     # touch docker-compose.yml
  ```
  + Fill out docker.compose.yml file on docker-host instance using this [gist](https://gist.github.com/Nklya/c2ca40a128758e2dc2244beb09caebe1) and run:
  
    $ docker-compose up -d
  + Ensure that GCP instance up and running with Gitlab web interface http://<your-vm-ip>
  
  ## App testing:
  + Clone current repository to your environment
  + Ensure that GCP instance up and running with Gitlab web interface http://<your-vm-ip>
  + Using SSH to docker-host run this [gist](https://gist.githubusercontent.com/SergeyKa-cmd/bc4035b974b0e07b62397dab2ad1bd2a/raw/7e422223e5fcaf82b389f6a856e1eadf70c64c04/Gitlab%2520Runner%2520registration) in terminal for Runner registration
  + Second step to register Gitlab Runner interactively in terminal:
  
  ```root@gitlab-ci:~# docker exec -it gitlab-runner gitlab-runner register --run-untagged --locked=false
     Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
      http://<YOUR-VM-IP>/
      Please enter the gitlab-ci token for this runner:
      <TOKEN>
      Please enter the gitlab-ci description for this runner:
      [38689f5588fe]: my-runner
      Please enter the gitlab-ci tags for this runner (comma separated):
      linux,xenial,ubuntu,docker
      Please enter the executor:
      docker
      Please enter the default Docker image (e.g. ruby:2.1):
      alpine:latest
      Runner registered successfully.
  ```    
  + All further manipulations were done in .gitlab-ci.yml file where added and described stages:
  ```stages:
    - build
    - test
    - review
    - stage
    - production
   ```
  ## Additional task tips:
   + For reddit app implementation used this [Automatically build and push Docker images using GitLab CI manual](https://angristan.xyz/build-push-docker-images-gitlab-ci/) where decribed how to use DockerHub credentials in Gitlab:
   ![Alt text](/home/sergeyka/Desktop/variables.png?raw=true "Title")
   + Placed code to .gitlab-ci.yml file in build stage section
   + Gitlab Runner configuration uses the .toml file [Gitlab Documentation](https://docs.gitlab.com/runner/configuration/advanced-configuration.html) for multitasking 
   + For automation deployment of numerous Gitlab Ci Runner we're used config.toml.example file (config.toml was excluded with .gitignore due to secret information) related to this [Manual](https://habr.com/en/post/449910/)
   + For Chat Ops implementation (Gitlab+Slack) we're used this [Simple manual](https://docs.gitlab.com/ee/user/project/integrations/slack.html)
   ___________________________________________________________________________________________________________________
   ## 6. Monitoring: Prometheus configuring and deployment
  ### Main issue: Deployment Prometheus and monitoring implementation for Reddit microservices
  ### Additional task:  Using custom exporters for metrics capturing & Prepare Makefile for services automation deployment
  ## System prerequisites:
  + Prepare firewall rules in GCE using terminal on local host:
  
    $ gcloud compute firewall-rules create prometheus-default --allow tcp:9090
    
    $ gcloud compute firewall-rules create puma-default --allow tcp:9292
  + Use current running GCE instance using command:
  
    $ eval $(docker-machine env docker-host)
    
    $ docker-machine ssh docker-host
  ## App testing:
  + Clone current repository to your environment
  + Fetch docker images from current GitHub repository using ```make``` command in your terminal
  + Ensure that GCP instance with Prometheus up and running on http://your-vm-ip:9090
  ## Additional task tips:
   + For preparing Makefile use this [tutorial](http://rus-linux.net/nlib.php?name=/MyLDP/algol/gnu_make/gnu_make_3-79_russian_manual.html#SEC33)
   + Information regarding how to use [Google Cloudprober](https://hub.docker.com/r/cloudprober/cloudprober)
   + [Bitnami MongoDB exporter on DockerHub](https://hub.docker.com/r/bitnami/mongodb-exporter)
   __________________________________________________________________________________________________________________________
   
