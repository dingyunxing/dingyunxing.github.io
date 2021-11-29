---
layout: post
title: How to use Python FastAPI and NGINX to deploy a web app  with docker compose
category: Tutorial
tags: [python, nginx, docker]
---

### Background
[FastAPI](https://fastapi.tiangolo.com/) is a high performance web framework in Python 3.6+ that booming very fast recently. While [NGINX](https://www.nginx.com/) is one of the most popular open source load balancer that been used widely as a reverse proxy in front of the app.
<!-- more -->

This article will show the steps to build a `Hello World` web app that hosting locally and set up a nginx server with it. All services will be deployed with `docker compose`. For more details about docker, please refer to [here](https://dingyunxing.github.io/tutorial/2021/10/07/mysql-docker/)


### Steps

### 1. Install Python and Poetry
There are bunch of ways to install Python. The official tutorial is from [here](https://www.python.org/downloads/) or with some third party tools such as Anaconda(https://www.anaconda.com/).

There are also bunch of ways to manage the virtual environment in Python. Here we highly recommand a tool called [poetry](https://python-poetry.org/). 

**Notes**: 
Please just skip this part if you have already got these ready on your local machine.


### 2. Create a new project with poetry

Start a new project with the command below. You can replace the `hello-nginx-app` with any other app name you like.

```
poetry new hello-nginx-app
```

Then add the library FastAPI with
```
poetry add fastapi
```

Now we have the place to start coding!


### 3. Write the app
First of all, as the app will be containerize with Docker, so we will add a `Dockerfile` in side the project. And the project structure looks like:
```
hello-nginx-app
├── pyproject.toml
├── README.rst
├── Dockerfile
├── hello-nginx-app
│   └── __init__.py
└── tests
    ├── __init__.py
    └── hello-nginx-app.py
```
Then we can create a `main.py` under `hello-nginx-app/`

And inside the main.py. add the code below:

```
from fastapi import FastAPI

app = FastAPI()

@app.get("/", status_code=200)
async def hello():
    return {"msg": "Hello World"}

```

Then in the Dockerfile, copy the code below:
```
FROM python:3.9.5-slim-buster

ARG app_root=/app
WORKDIR /app

COPY ./poetry.lock ./pyproject.toml ${app_root}/
COPY ./hello-nginx-app ${app_root}/

RUN pip install poetry==1.1.3
RUN poetry install && poetry cache clear --all -n pypi
CMD ["poetry", "run", "uvicorn", "main:app", "--host", "0.0.0.0"]

```

### 4. Prepare the NGINX

Then, we can create another folder for nginx, this folder will then be put together with the folder `hello-nginx-app` in another folder say `fastapi-nginx-compose` or whatever name you like.

The nginx folder's structures will be
```
nginx
├── nginx.conf
├── Dockerfile
```

Here, the `nginx.conf` defines the configuration of the nginx while the dockerfile will strat a docker

in the nginx.conf, add code
```
events {
    worker_connections 100;
}

http {
    resolver 127.0.0.11 ipv6=off;
    server{
        listen 80;
        location / {
            proxy_pass http://api:8000;
        }
    }
}
```

and in the Dockerfile, add

```
FROM nginx

COPY nginx.conf /etc/nginx/nginx.conf
```

### 5. Docker Compose the App with Nginx

Now we can put the app folder and nginx folder together in another parent folder, and create a `docker-compose.yaml` file for setting up.

```
fastapi-nginx-compose
├── hello-nginx-app/
├── nginx/
├── docker-compose.yaml
```

The code in docker-compose.yaml

```
version: "3.9"

services:
    web:
        build: nginx
        ports:
          - 80:80
          - 443:443
        depends_on:
          - api
    api:
      build: airflow-token-api
      expose:
        - 8000
      ports:
        - 8000:8000
```

### 6. Run the app

Now it's time to test if it's running good:

At the parent folder, run
```
docker-compose up
```

Then you can open a browser and enter `localhost`

There should be printing out the `{"msg": "Hello World"}`

