# Infrastructures for Big Data Processing (BDP2): “Mid-term” Review

This repository contains the necessary information and files generated to complete the mid-term review for the Big Data Processing (BDP2) course of the Master's Degree in Bioinformatics of the Università di Bologna, course 2022-2023.

This review consists of (1) extending an exisitng Docker image and (2) creating an application stack using `docker-compose`, in an already created AWS virtual machine.

Written by Laia Torres Masdeu.

**Table of Contents**
-  [0. Requirements and first steps](#0-requirements-and-first-steps)
    - [0.1. Requirements](#01-requirements)
    - [0.2. First steps](#02-first-steps)
- [1. Extend the image](#1-extend-the-image)
- [2. Create an application stack](#2-create-an-application-stack)
- [3. Additional tasks](#3-additional-tasks)
    - [3.1. Try to implement `https` access to Jupyter](#31-try-to-implement-https-access-to-jupyter)
    - [3.2. Show if and how it works on your laptop in the same way it works on VM1](#32-show-if-and-how-it-works-on-your-laptop-in-the-same-way-it-works-on-vm1)

## 0. Requirements and first steps
#### 0.1. Requirements
To be able to replicate this review, ensure you have installed in your virtual machine the following programs:
- `docker`: to run containers (`run`), track running containers (`ps`)  and inspect already created images (`images`)
- `git`: to track changes in your directory and to clone it to/from GitHub (`push`/`pull`)
- `docker-compose`: to create the application stack

#### 0.2. First steps
Before starting the exercise, as we are working with AWS, log in to the virtual machine of preference (in this case, VM1) through `ssh`. Remember to first go to the directory where our key is stored.
```
cd /Users/laiatorres/UNIBO/BDP2
ssh -i "bdp2.pem" ubuntu@ec2-<VM1_public_IP>.compute-1.amazonaws.com
```

Then, create a new directory, from which we will be working, and go to it:
```
mkdir -p ~/review
cd ~/review
```

In this first pre-step we will run the (default) Jupyter and Redis containers, and set them in the same network (bridge, in this case), as we want to be able to access the Redis server from within the Jupyter notebook. So, first create the `bdp2-net` bridge
```
docker network create bdp2-net
```
and then the Redis and Jupyter containers
```
docker run -d --rm --name my_redis -v ~/review:/data --network bdp2-net --user 1000 redis redis-server
docker run -d --rm --name my_jupyter -v ~/review:/home/jovyan -p 80:8888 --network bdp2-net -e JUPYTER_ENABLE_LAB=yes -e JUPYTER_TOKEN="<insert_your_token>" --user root -e CHOWN_HOME=yes -e CHOWN_HOME_OPTS="-R" jupyter/minimal-notebook
```
, where:
* `-d`: run the container in the background
* `--rm`: automatically remove the volume when it exits
* `--name`: name of the container (`my_redis` and `my_jupyter`)
* `-v`: Docker volumes, to mount a local directory into the container (where the path before the `:` is the path to the local directory, `~/review` in this case)
* `--network`: the network to which we are connecting both containers
* `-p`: port mapping (`<docker_host_port>:<container_port>`)
* `-e`: environment variable. In this case, it is important to change the `JUPYTER_TOKEN`, which is the password we will use to enter Jupyter Notebook
* `redis` and `jupyter/minimal-notebook`: the Docker images used to build the containers

Once both containers are up (check with `docker ps`), connect to Jupyter, by going to the URL [http://<VM1_public_IP>:80](http://<VM1_public_IP>:80) on our browser. Before this, it is important to open port 80 for your laptop: go to the security group for VM1 and Edit inbound rules:

Once it loads, it will ask for a password/token, which is the string that we have written after the `JUPYTER_TOKEN` environment variable in the `docker run` command for Jupyter. 

Start a Jupyter notebook. If we try to run Redis `import redis`, it will give us an error, as the Redis module is not installed by default. By running `!pip install redis` in the same notebook, it will install the module, and will now be able to work. However, as containers are effimeral, if we stop it and re-run it again, we will have to re-install the Redis module. Therefore, we want to create a cutom image that already has this module incorporated.


## 1. Extend the image
The aim of this first step is to create, from an already existing Jupyter Notebook Docker image (`network create bdp2-net`) a custom image that includes the `redis` module, using `git` to track the changes and push them to GitHub and to automatically build this custom image on DockerHub (through the use of *GitHub actions*). 

As we are creating a new Jupyter image, the first thing to do is to stop `my_jupyter`:
```
docker stop my_jupyter 
```

As mentioned, we want to track all of the changes in the files of our directory using `git`. To do so, go to your browser, log in to GitHub and create a new public repo. Then clone it to the VM1:

```
git clone https://github.com/<your_username>/<your_repo_name>.git
```
and move into the VM1 repo directory:
```
cd bdp2-review/
```

In this directory, we want to add two directories, to have this structure:
- `.`: main directory, where we will put the `README.md` file and the `docker-compose.yml` file (step 2)
- `./docker`: where we will write and save the `Dockerfile`to create the custom image
- `./work`: where we will store the Jupyter notebooks

To create these directories:
```
mkdir docker
mkdir work
```

Now, we will create the GitHub Action that will build the custom image from the Dockerfile and will push it to DockerHub. To do so, go inside your GitHub repo, click on Actions, and search for "Docker image By GitHub Actions" select it (click on "Configure"), write the following and click on "Commit changes...":
```
name: Docker Image Jupyter+redis

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:

  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    - name: Build the Docker image
      run: docker build . --file docker/Dockerfile --tag <image_name>
    - name: Push the image to DockerHub
      run: docker login -u <DockerHub_username> -p${{ secrets.DOCKER_TOKEN }} && docker push <image_name>
```
What does this file do? Line by line:
* 1: the name of the workflow (`Docker Image Jupyter+redis`)
* 3-7: when will this action be triggered (every time you push or pull the "main" branch)
* 9-20: build the Docker image and push it to DockerHub:
    * `--file`: path and name of the Dockerfile that contains the information to build the image
    * `--tag`: name of the image (in my case, `torresmasdeu/ltm_jupyter`)
    * `-u`: username of DockerHub 
    * `-p`: DockerHub token (NOT the password)

To retrieve the DockerHub token, log in to your DockerHub account, and go to "Account Settings" (by clicking on your username). Then, click on "Security", and then on "New Access Token". Give it a name (i.e. "Token for GitHub"), and select "Read, Write, Delete" as access permissions. Lastly, click on "Generate" and copy the personal access token (save it somewhere too, as you will not be able to acces it once you exit).

Now, go to GitHub, and go to the repo "Settings". Scroll to "Secrets and variables", "Actions" and lastly click on "New repository secret". Call it "DOCKER_TOKEN" (or whatever name is after `-p${{ secrets.` on the Docker image file) and paste the token you just created on DockerHub. Click on "Add secret" and verify it is listed as a "Repository secret".

Now the `docker-image.yml` file is created and ready to be run whenever any commits are done to the repo. However, it is still not creating the image, as the `Dockerfile` is missing.

Before creating the `Dockerfile`, we have to ensure to pull the new addition that we have on the GitHub repo to the local repo (that is, the `docker-image.yml` file):
```
git pull
```

It will ask you for your GitHub username and password. However, for security purposes, the password is not the one you use to connect to GitHub, but a GitHub Personal Access Token. This can be retrieved in the general "Settings" (by clicking on your profile photo),  “<> Developer settings” and heading to "Personal access tokens". Then, click on “Generate new token” , “Generate new token (classic)” and give it a name (i.e. "BDP2 GitHub token"), select a desired expiration date, and in "scopes" select repo. Lastly, click on "Generate token". As before, save the token, as you will not be able to acces it once you exit.

Check that the `docker-image.yml` file is now visible in your local repo (under `.git/workflows/` directory). 

Now we can generate the `Dockerfile`:
```
vi docker/Dockerfile
```

In this `Dockerfile` we want to write the commands to generate the custom image, that adds the Redis module inside the `jupyter/minimal-notebook` Docker image. It is as follows:
```
FROM jupyter/minimal-notebook
RUN pip install redis
```

Now add the `Dockerfile` to `git`, commit the changes and push them to the GitHub repo:
```
git add docker/
git commit -m "Commit Dockerfile"
git push -u origin main
```

**Important**: the repo is pushed to the `main` branch, and not the `master`, because in the `docker-image.yml` file we wrote that it was to be executed when pushing to the `main` branch. Moreover, said file is saved in the `main` branch, so if we were to push things in the `master` branch, we would have two branches that could not interact with each other.

Now, if no additional problems arise, the action should have been executed successfully. Check so by going to the "Actions" tab and seeing a green dot next to the last commit message you made. Moreover, go to your DockerHub account and verify that the new image is there:
![dockerhub](https://github.com/torresmasdeu/bdp2-review/assets/98785167/cbafe823-1c59-45e1-9ef1-67a2eec5cdcb)

Now, run the new custom image, and ensure that the Jupyter notebooks are written in the `work` directory:
```
docker run -d --rm --name my_redis_jupyter -v ~/review/bdp2-review/work:/home/jovyan -p 80:8888 --network bdp2-net -e JUPYTER_ENABLE_LAB=yes -e JUPYTER_TOKEN="bdp2_ltm" --user root -e CHOWN_HOME=yes -e CHOWN_HOME_OPTS="-R" torresmasdeu/ltm_jupyter
```

Some changes with respect to the previous `docker run`:
- The container is now attached to the `work` directory (`-v ~/review/bdp2-review/work:/home/jovyan`)
- The Docker image from which we now run the container is not the original one, `jupyter/minimal-notebook`, but the one we created using the action (in my case, `torresmasdeu/ltm_jupyter`). 

Once it is running, open Jupyter Notebooks in your browser, [http://<VM1_public_IP>:80](http://<VM1_public_IP>:80), and create a new notebook (`dir_test.ipynb`). As before, verify that the `redis` module is available in the Jupyter container: you should run the `import redis` command with no error messages.
![jupyter_run](https://github.com/torresmasdeu/bdp2-review/assets/98785167/53746cb8-b302-4687-bd7e-8305b1e9b645)

We can ensure that the connection with Redis is established by running a simple Redis command. 

**Note**: when connecting to the Redis server (`redis.Redis(host='my_redis')`), ensure that the string after `host=` is the name of your Redis container.

Lastly, save this file (`dir_test.ipynb`) and check that it is stored in the `work` directory:
![work_dir_run](https://github.com/torresmasdeu/bdp2-review/assets/98785167/4e318248-2539-4ebb-8d9b-24f65bdda4c5)

Before committing all, create the `.gitignore` file, that will include the names of any files on the local repo that we do not want to commit to `git`. In this case, add the `.ipynb_checkpoints` file, to avoid `git` complaining about “Notebook Checkpoints” not being tracked.
``` 
vi .gitignore
git add * 
git commit -a -m "Commit after docker run successful command"
git push
docker ps
```
Remember to run `git status` to check that all the files have been added to `git`.
And lastly, stop both containers:
```
docker stop my_redis
docker stop my_redis_jupyter
```

Run `docker ps` to ensure that no container is running.

## 2. Create an application stack
The aim of this second step is to build an application stack using `docker-compose` to run 3 containers simultaneously: the Redis image, the custom Jupyter image and the Portainer image. The names of the images can be found by running `docker images`:
![docker_images](https://github.com/torresmasdeu/bdp2-review/assets/98785167/35672ba2-ac73-4186-a9f4-b79fdad857a3)

All the mount points specified in the `docker run` commands have to be listed in the `docker-compose.yml` file. Let's remember these three commands:
```
#redis
docker run -d --rm --name my_redis -v ~/review:/data --network bdp2-net --user 1000 redis redis-server

#custom jupyter
docker run -d --rm --name my_redis_jupyter -v ~/review/bdp2-review/work:/home/jovyan -p 80:8888 --network bdp2-net -e JUPYTER_ENABLE_LAB=yes -e JUPYTER_TOKEN="bdp2_ltm" --user root -e CHOWN_HOME=yes -e CHOWN_HOME_OPTS="-R" torresmasdeu/ltm_jupyter

#portainer
docker run -d --rm -p 8000:8000 -p 443:9443 --name=portainer -v
/var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```

The building of the containers is done under `services`.

This is the parallelism between the `docker run` flags and the `docker-compose.yml` commands:
* Under `services:` we have three different levels, one for each container. The names of these levels are the names that will be used to name the container (before under the `--name` flag)
* `image`: the name of the image (unflagged in `docker run`)
* `ports`: `-p` flag
* `volumes`: `-v` flags. Important: if one of these flags links a volume to the container, the volume name has to be specified outside `services`, under `volumes` (see the case of the volume `portainer_data`)
* `networks`: as we want the three containers to be able to communicate between each other, we have to put them under the same network. As with the case of volumes, the name of the network(s) in use has to be specified in another section outside services, `networks`
* `environment`: `-e` flag. In the case of the password needed to connect to Jupyter Notebooks, `JUPYTER_TOKEN`, we do not want it to be visible to anyone. This is why I decided to set an environment variable, called `JUPYTER_PW`, that will look for my password in the `.env` file I have created in the same directory as the `docker-compose.yml` file. **Note**: to keep from adding this file to `git`, add the file name (`.env`) to the `.gitignore` file
* `user`: `--user` tag

Taking all of this into consideration, this is the resulting `docker-compose.yml` file:
```
version: "3"
services:
  portainer:
    image: portainer/portainer-ce
    ports:
      - 8000:8000
      - 443:9443
    volumes:
      - portainer_data:/data
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - bdp2-net

  jupyter:
    image: torresmasdeu/ltm_jupyter
    environment:
      - JUPYTER_ENABLE_LAB=yes 
      - JUPYTER_TOKEN=${JUPYTER_PW} 
      - CHOWN_HOME=yes 
      - CHOWN_HOME_OPTS=-R
    user: root
    ports:
      - 80:8888
    volumes: 
      - ~/review/bdp2-review/work:/home/jovyan
    networks: 
      - bdp2-net

  redis:
    image: redis
    user: "1000"
    volumes:
      - ~/review:/data
    networks: 
      - bdp2-net
networks: 
  bdp2-net:

volumes:
  portainer_data:
```

Run, from the directory where the `docker-compose.yml` file is, the `docker-compose` command to create the application stack:
```
docker-compose up -d
```

Check that the containers are running by running `docker ps`. 

To stop the stack, run `docker-compose down`:
![docker_compose](https://github.com/torresmasdeu/bdp2-review/assets/98785167/9de99b4d-2aa4-4e8e-9a47-2a49295f7a65)

**Note**: regarding *bridge networking*, despite having named the network in `docker-compose.yml` with the same name as the network I created (`bdp2-net`), when running `docker-compose up -d`, it creates a new bridge network, `bdp2-review_bdp2-net`:
![compose_bridge](https://github.com/torresmasdeu/bdp2-review/assets/98785167/16e47643-ff19-46d1-abf9-9b1510e02c5c)

To check this, I removed `bdp2-net`, and `docker-compose up -d` worked with no problems:
![remove_network](https://github.com/torresmasdeu/bdp2-review/assets/98785167/5a87b1c7-f1f1-47e6-be1e-99799177bdfc)

Actually, we can clearly see that, asides from creating the three containers, `docker-compose up -d` has also created the network.

Before checking with portainer that the containers are working successfully, remember to open port 443 (of the portainer container) for your laptop to be able to access the portainer website, [https://<VM1_public_IP>](https://<VM1_public_IP>). Log in to portainer (choose whatever admin password you want when connecting for the first time), and click on "Get started", to visualise your local compartment. This is what the containers on my local environment looked like:
![portainer](https://github.com/torresmasdeu/bdp2-review/assets/98785167/69a2f04b-14e1-4b6b-8717-1b18a55cb322)

Lastly, check that the Redis server and Jupyter are working by connecting to Jupyter Notebooks ([http://<VM1_public_IP>:80](http://<VM1_public_IP>:80)), signing in with your password (saved in the `.env` file) and do as before. Remember to change the string after `host=` to the name of your actual Redis container (check `docker ps` for it):
![jupyter_compose](https://github.com/torresmasdeu/bdp2-review/assets/98785167/4c6a7575-24b6-4165-8898-1883a89709a8)

Save the file (`dir_test_compose.ipynb`) and check that it is stored in the `work` directory:
![work_dir_comp](https://github.com/torresmasdeu/bdp2-review/assets/98785167/f63ad3aa-b9ed-492a-a040-5df59679805c)

## 3. Additional tasks

#### 3.1. Try to implement `https` access to Jupyter
To access Jupyter through `https` we need to add port 443 to Jupyter Notebook port (8888) to the list of ports for the Jupyter container, that is, add `- 443:8888` to the `docker-compose.yml` file. 

However, this returns an error, stating that port 443 is already allocated (remember that portainer uses it): 
![https_compose](https://github.com/torresmasdeu/bdp2-review/assets/98785167/78332ab7-f1d6-4421-a206-9d9dab638ff3)

Therefore I decided to try it by running Docker with `docker run`, starting the Redis and custom Jupyter images:
```
docker run -d --rm --name my_redis -v ~/review:/data --network bdp2-net --user 1000 redis redis-server

docker run -d --rm --name my_redis_jupyter -v ~/review/bdp2-review/work:/home/jovyan -p 80:8888 -p 443:8888 --network bdp2-net -e JUPYTER_ENABLE_LAB=yes -e JUPYTER_TOKEN="bdp2_ltm" --user root -e CHOWN_HOME=yes -e CHOWN_HOME_OPTS="-R" torresmasdeu/ltm_jupyter
```
The containers are created and run with no problems, but when trying to access Jupyter with `https` it does not find the server, even if port 443 is open to my laptop. [Searching this issue](https://leangaurav.medium.com/quick-jupyterhub-setup-docker-nginx-https-letsencrypt-aws-cloud-57d1afa5c253), I have found in multiple sites that to access Jupyter Notebook with https you should enable SSL certificate in the Jupyter notebook server.

#### 3.2. Show if and how it works on your laptop in the same way it works on VM1
To make this, I first started Docker Desktop.

Then, I decided to create a directory called `review` in the same way as I did in VM1. This way, I could use the original `docker-compose.yml` file without modifying it.
```
mkdir -p ~/review
cd ~/review
```

Then, I cloned the repo in this directory and moved in the local repo:
```
git clone https://github.com/torresmasdeu/bdp2-review.git
cd bdp2-review/
```

Before running `docker_compose`, I created a `.env` file to store the `JUPYTER_PW` password, and started the application stack:
```
docker-compose up -d
```

To check that Jupyter was working, I connected to http://0.0.0.0:80 and put in my password. I imported Redis and connected to my newly created Redis container (running `docker ps` to check its name), and saved the file (`dir_test_my_laptop.ipynb`):
![jupyter_laptop](https://github.com/torresmasdeu/bdp2-review/assets/98785167/9235efd3-4c2c-4fa1-836a-e67692007157)

I then checked that the file was stored in the `work` directory:
![work_dir_laptop](https://github.com/torresmasdeu/bdp2-review/assets/98785167/961cd6a5-ee0e-4331-8e90-9b399e825a4b)

Lastly, I checked that Portainer was working. I connected to https://0.0.0.0:443, put a new admin password and clicked on "Get started". This is what the containers on my local environment looked like:
![portainer_laptop](https://github.com/torresmasdeu/bdp2-review/assets/98785167/b698e966-855b-4a84-9668-379b6373d14b)
