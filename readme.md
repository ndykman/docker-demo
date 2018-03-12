# Introduction

Welcome to a very simple walkthrough of some docker basics on Linux and Windows. At the end, you will have run some basic containers and debugged a basic ASP .Net Core application using Docker and Visual Studio. 

## Requirements

Windows 10 with the Fall Creators Update applied (latest updates are recommended)
Docker Community Edition for Windows (https://www.docker.com/community-edition)
Visual Studio 2017 Community Edition (for the ASP .Net Core demo). Make sure to update to at least version 15.6.x. 

### Starting out.

If you move this directory to C:\git, it will simplify the walkthrough a great deal. If not, replace C:\git with the parent directory where it appears. 

# Stage 1: Docker Basics

Find the docker UI icon (it looks like a whale) and right click it. This should present a context menu.

Make sure you are running in Linux container mode. You should an option to "Switch to Windows Containers".
If, just select the "Switch to Linux Containers" on the context menu. 

To run these demos, you will need to share the drive this file is located on. Right Click on the docker icon, select the settings menu option. Select Shared Drives in the UI and share the appropriate drive. If you are in Windows container mode, you will not see this option as it is not needed.

Also, it is highly suggested you disable experimental features as they may cause instability. 

## Step 1.1: Opening Powershell

Run Windows Powershell as we will be using the command line in the rest of this walkthrough.

## Step 1.2: Pull an image

Run the following command in the powershell window

    docker pull nginx

This should download the image and make it ready to use in docker containers.

A docker image is a way to distribute the files needed to run a specific operating system or software as a container instance.

Now, run the command 

    docker ps

This shows what containers are running. There should be no containers running.
 
## Step 1.3: Run a container

Run the following command in the powershell window. (from now on, assume all commands are run in powershell)

    docker run -d -v C:\git\docker-demo\linux-html:/usr/share/nginx/html -p 8001:80 --name nginx-shared nginx

Load a browser and open up http://localhost:8001 and you should see a basic html page greeting you. 

Let us walk through each option above.

    -d: run the container as detached (in the background). 

By default, the process is run in the foreground and console output from the container is redirected to the console the command was run in.

    -v: map a host volume to the container. 

This allows the container to access a host directory as if it was on the container itself. 

    -p: map a host port to the container port.

This sets up redirect of a host network port to the container port (there is a gotcha with Windows containers, see Step 3). In this case, all traffic from localhost:8001 will go to port 80 on the container. 

    --name: Name a container instance 

This assigns a name to the container instance that can be used with other docker commands. If not provided, a name is generated. It is a good practice to name all your containers, as it makes stopping and removing them easier. 

### Experiment (optional): Change index

Go to the linux-html directory and edit the index.html file. Reload the above page. What do you think will happen? What actually happens?

## Step 1.4: Stopping a container

Run the following command  

    docker stop nginx-shared

This will stop the running container. Then, run

    docker rm nginx-shared

This will remove the running container. Containers are not by default removed when run, as parts of them can be mounted after they exit. This is mainly useful for advanced post-mortem analysis of why a container stopped by accessing logs and other files.

# Step 2: Dockerfile basics. 

We will now transition to the use of Dockerfiles to create new images from a base image. 

Change to the linux-nginx directory and open the Dockerfile. 

Notice the following line:

    COPY index.html /usr/share/nginx/html

This says to copy the a file (or the contents of an entire directory) to a directory on the host. 

To create a new image, we must build from the docker file. To better track images, we give it a tag. Run the following command

    docker build -t docker-demo/nginx-1 .

Then, make the build worked by running the image. 

    docker run -d -p 8002:80 --name ngnix-static docker-demo/nginx-1

You should now be able to browse http://localhost:8002 and see a slightly different web page.

### Experiment (optional): Change the index.html File

In this directory, change the contents of the index.html file. What should happen when you reload the page? What actually happens? Why is this the case? 

# Step 3: Dockers and Windows Containers

Use the Docker UI to switch to Windows containers. If you tire of using the UI for this, you can run the following in Powershell.

    & 'C:\Program Files\Docker\Docker\DockerCli.exe' -SwitchDaemon

Now, we are in windows container mode. Now, run 

    docker images

This command shows all the images on the machine. Note, there are no images that we created above listed. This is because we are in windows container mode and we haven't gotten a windows image yet. Now run

    docker pull microsoft/iis:latest

This will download a image that can run Internet Information Service. Beware, this is a big file and it will take some time to download. 

When that is done, change to the windows-iis directory and review the Dockerfile. Then, run:

    docker build -t docker-demo/iis .

To build an image based on the microsoft image. Then, run:

    docker run -d -p 8003:80 --name dockerdemo-iis-1 docker-demo/iis

Then try to browse http://localhost:8003. In some cases, this will not work due to a Windows 10 bug in the underlying network stack. This is slated to be fixed in the next update of Windows 10. In the meantime, the workaround is browse using the container IP address. To find this, run

    docker inspect --format '{{ .NetworkSettings.Networks.nat.IPAddress }}' dockerdemo-iis-1

Docker inspect is a command to get the details of a running container. When you use the container IP address, don't use the 8003 port, as it is not in use on the container. Just use port 80. 

Another option is use the host DNS or IP address and the redirected port. If you use this option, use the host mapped port. In this example, this is 8003.

# Step 3.1 (optional): Run the Default IIS Image

Run the following:

    docker run -d -p 8004:80 --name microsoft-iis-1 microsoft/iis

This will take a bit of time, but will run the default iis image. If you browse to the appropriate location as you did above (but using 8004 instead of 8003), you should see the default IIS page. 

You can experiment with volume mapping, but in Windows, permissions are more complex. 

# Step 4: Docker Compose

First, switch back to docker linux container mode. Run docker images to verify.

Then change to the linux-compose directory. Open the docker-compose.yml file. Note the lines:

    volumes:
      - "C:\\Git\\docker-demo\\linux-html:/usr/share/nginx/html"

Make sure this matches the path to the docker demo directory, or this section will fail. 

We can't touch on all the details here (see the docker documentation), but the docker-compose.yml file allows for multiple containers to built and run together. 

Run: 

    docker-compose build

To create the needed images. Then run:

    docker-compose up -d 

You should now be able to browse to http://localhost:8005 and http://localhost:8006 to see two different web pages. The -d option runs the containers in detached mode (background). Note, if not use, docker will redirect the console output for all the containers to the console. This can be useful. 

Browse to http://localhost:8005 and http://localhost:8006 to see the running web pages. 

Then run:

    docker-compose down 

This will stop and remove both containers that were just ran. 

The goal of docker compose is to allow for a set of containers to be run together. This allows for more complex scenarios to be easily supported. 

# Step 5 (optional): ASP .Net Core example 

Change to the DockerASPNet directory and open the DockerASPNet.sln file. Review the project structure and the docker-compose and Dockerfile files in the projects. Go to the Controllers/HomeController.cs and set a breakpoint on the 

    return View("Index");

line. Debug the project. It should open a browser and stop on the line. 

Note, we did not switch back to Windows container mode (if this doesn't work, make sure you aren't in windows container mode first) but still are in Linux container mode. Despite this, we can run and debug the container. The project was setup to use Linux containers for running and debugging. Unless a windows container is needed, this is often the recommended default. 

# Step 6: Docker Hub

By default, when looking for images, docker will look in the docker hub repository (hub.docker.com). It is useful to browse that site to see all the images that are available. Also, you should be aware of the docker store, a place to find officially supported images from vendors. 

# Docker Future 

Currently, Docker and Microsoft are working to make the use of linux and windows containers much more streamlined, a feature that is named LCOW (Linux Containers on Windows), when this is finished, the overall docker experience will change. 

# Summary

This is a basic overview of working with Docker in the Windows 10 environment. This only scratches the surface of what is possible with docker and does not cover many topics, but does cover many basic operations. 