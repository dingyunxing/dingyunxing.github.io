---
layout: post
title: How to install MySQL and MySQL workbench with docker compose
category: Tutorial
tags: [MySQL, docker]
---

### Background
MySQL is one of the most popular open source relational database management system (RDBMS) thus a good tool for learning SQL. This artical shows how to quickly implement a MySQL env locally with docker-compose and visit it by a GUI called MySQL workbench.
<!-- more -->

### Steps
#### 1. Install Docker
Docker is a software that use OS-level virtualization to deliver software in packages called containers.

**Notes**: 
Please just skip this part if you have already got Docker on your local machine.

For different OS, please go to this [page](https://docs.docker.com/get-docker/) for geting the latest docker install instructions.

- Windows/Mac OS

Docker offer Docker Desktop for Windows or Mac OS so it's only need to download the installation package and install as normal software on your machine. More instructions here for [Windows](https://docs.docker.com/desktop/windows/install/) or [Mac](https://docs.docker.com/desktop/mac/install/)

- Linux

There are bunch of different version of Linux, [here](https://docs.docker.com/engine/install/ubuntu/) is an example on how to install Docker on Ubuntu with bash commands.

After install, open a Terminal/git bash/powershell or whatever command line tools you perfer and type
```
docker version
```
If got some info then it means the docker has been successfully installed.


#### 2. Start MySQL with docker-compose

Create a new folder (can name `mysql-docker` for example) , inside the folder, create a new file called `docker-compose.yml`. Copy paste the code below into this .yml file
```
version: '3.3'
services:
  db:
    image: mysql:latest
    restart: always
    environment:
      MYSQL_DATABASE: 'db'
      # So you don't have to use root, but you can if you like
      MYSQL_USER: 'username'
      # You can use whatever password you like
      MYSQL_PASSWORD: 'password'
      # Password for root access
      MYSQL_ROOT_PASSWORD: 'password'
    ports:
      # <Port exposed> : < MySQL Port running inside container>
      - '3306:3306'
    expose:
      # Opens port 3306 on the container
      - '3306'
      # Where our data will be persisted
    volumes:
      - my-db:/var/lib/mysql
# Names our volume
volumes:
  my-db:
```
Then within the folder, run command
```
docker compose up
```
Now the mysql have been started. To test, just open a new terminal window and type
```
docker ps
```


#### 3. Install the MySQL workbench
Terminal is not that user-friendly for most of the people so we need a GUI to connect to the MySQL service.

Download the latest version of the MySQL Workbench [here](https://dev.mysql.com/downloads/workbench/). Select the right OS version and download and install just similar to the Docker Desktop

Start the Workbench afterwards and make a new connection like this:
![](https://miro.medium.com/max/4800/1*VcfoGGvE6rtXsaykGxpvCw.png)

![](https://miro.medium.com/max/4800/1*bcahaA-dWpHd16E-H5kGHg.png)

Input password and click “OK” to connect.