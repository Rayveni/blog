# Asynchronous Tasks with Flask,RQ and Redis in Docker
![enter image description here](https://github.com/Rayveni/blog/blob/main/articles/flask%20redis/img/flask_redis_rq.jpg?raw=true)
This tutorial will walk you through deploying up a basic Flask app using background jobs with the help of Docker containers.
 Code template can be downloaded from [github/Rayveni](https://github.com/Rayveni/blog/tree/main/flask%20redis).
## Table of Contents

 - Docker architecture
 - Prerequisite
 - How to start containters and app
 - Project Structure
 -  Docker-compose.yml structure
 - worker and server
 - Flask Application
 -  Closing thoughts


**Docker container** is a sealed environment where you can run your application and all its dependencies. **Docker image** contains all the tools and libraries needed to run your application, including any necessary setup files.  
The images are generated from a set of files describing how the application container is run in production and development.

**Flask** is a lightweight Python-based micro web framework, which it easy to build web applications and REST APIs with its built-in HTTP request/response system.

**Redis** is an open source in-memory **data structure store,** used as a database, cache, message broker, and streaming engine.

**RQ** is a powerful python library for managing asynchronous task queues and workflows on top of Redis.

In this project, we will develop a simple queue to run background tasks using Redis as the message broker. The publisher (flask app) enqueues the task names and its arguments, and the subscriber (worker) executes the tasks.
![enter image description here](https://github.com/Rayveni/blog/blob/main/articles/flask%20redis/img/redis_queue.jpg?raw=true)
##  Docker architecture
![enter image description here](https://github.com/Rayveni/blog/blob/main/articles/flask%20redis/img/docker_design.jpg?raw=true)
First, we have to build a docker application with three containers:
-   Flask app image
-   Redis image
-   Redis queue listener image

For Redis we use pre-built image from **DockerHub**.
For the application and the worker, we create an image from Dockerfile.
 Using the same image for redis queue listener and flask application is a bit tricky, but it was done for simplicity. 
 
## Prerequisite
-   Install Docker Desktop

Visit [https://www.docker.com/get-started/](https://www.docker.com/get-started/) to download Docker Desktop for your system.
##  How to start containters and app
Perform next steps:

 1. Download example code  from [Github/Rayveni](https://github.com/Rayveni/blog/tree/main/flask%20redis)
 2. Navigate to project folder in terminal and execute:
 ![enter image description here](https://raw.githubusercontent.com/Rayveni/blog/main/articles/flask%20redis/img/terminal-min.png)
     ```bash
     docker-compose --env-file .env build
     ```
     This command creates docker images for the application.
 4. To run container with application built from previous step images :
	   ```bash
		 docker-compose --env-file .env up -d
	```
 5.  In browser, navigate to **localhost:5000**:
![enter image description here](https://raw.githubusercontent.com/Rayveni/blog/main/articles/flask%20redis/img/app_screen.jpg)
 7. Run background tasks by submitting task parameters in the input form (in implemented background task, the input parameter is returned with a 60 second delay) and review the results (click the **Completed Tasks** link).
Explanation of the project's source code below.
 9.  To stop the container and the application:  
	   ```bash
	   docker-compose --env-file .env down
		``` 
 
 ## Project Structure 
 
 ```
flask redis
┣━[backend]
┃ ┣━[src]
┃ ┣━[tasks]
┃ ┣━.dockerignore
┃ ┣━Dockerfile
┃ ┣━requirements.txt
┃ ┣━server.py
┃ ┗━worker.py
┣━.env
┣━.gitignore
┣━docker-compose.yml
┗━readme.md
```

In this folder structure, the **backend** folder contains :
 1.   **src** folder— Flask application location and corresponding APIs.
 2.   **tasks** folder— The code for asynchronous tasks is stored here
 3.   server.py —HTTP server launcher (Gevent server in current example)
 4.   worker.py —  Redis queue listener and job executor
 5.   **Dockerfile** —  is a text file that contains all the commands that make up a Docker image (to keep things simple, I've created just one image for app and worker).
##  Docker-compose.yml structure
In the  **How to start containters and app** section   , the **env-file** argument has been applied to the docker-compose command.
**docker-compose** supports parameter expansion in environment files. In simple terms(`${..}`)  is replaced  with corresponding named parameters (in our project these parameters are stored in the **.env** file) when running docker-compose command.
This approach allows easily configure environment, container run in (e.g. prod/dev/test) .

https://gist.github.com/Rayveni/9f55f67af6dfe4442f228e9213dcea4e

 Following images created:

 - local_image/flask_rq - from **Dockerfile** in backend folder for our flask application
 - image for redis db -from official docker hub

**Dockerfile** is pretty straight forward(simple installation necessary python libs from requirements file):
```bash
FROM python:3.9.16-slim-bullseye
COPY requirements.txt /
RUN pip3 install --no-cache-dir -r /requirements.txt
WORKDIR /app
```
Worker(queue lister) and app server are  distinguished by **command:python worker.py** and **command:server.py** in docker-compose.yml.

There are 2 options for getting code into Docker:

 - Copy to docker image with help of Dockerfile, using copy command (same as ```COPY requirements.txt``` ) which is better way for prod environment , but it takes time to recreate image (need to run build with no cache option) each time the app code changes.
 - Mount the code from a directory of the host: a better way for application development since the changes are applied immediately.

In this project   **backend** folder  mounted for worker and app services. Note that in real project, worker code and app code should be independent.
## worker and server
https://gist.github.com/Rayveni/44690444d49bf881f0f658c94f072e52.js

 - **server.py** -launches gevent http server on docker virtual network ("0.0.0.0:80").
 To make it visible on the host in docker-compose (after replacement from .env), the following instruction is used:
	 ```bash
	 ports:
	- "5000:80"
	 ```
	 80 - is a docker virtual network port (you can use anyone) "0.0.0.0:**80**"+"5000:**80**"->localhost:5000
 - worker.py - start listener in redis queue(redis://redis:6379). There can be multiple queues (for example, you can design a queues for high/low/medium priority tasks ). Only one queue was used in our project: **default**.
## Flask Application
https://gist.github.com/Rayveni/b3fc00fe9dd068dbae055289e0c21a49
```
[src]  
┣━[static] 
┃     ┗━[css]
┃     ┃   ┗━mycss.css  
┃     ┗━docker_flask.png       
┣━[templates]
┃     ┗━index.html
┗━__init__.py
```
Our App has 2 endpoints:
 - "/"  -for index page (render  **index.html** from templates folder)
 - /finished_tasks-  returns a json with completed tasks info.

Post request from the **index.html**  calls the **__add_task_to_queue** function, which simply adds tasks to the **default** queue ( as we remember, the started worker listens to this queue).

Arguments:  'tasks.simple_task' and input form parameter are passed to **are__add_task_to_queue**.Good practice to pass string pointers to worker function instead passing instances of functions (python will try to pickle them to pass to worker and not always succeed in it),so better writhe independent from each other app and worker code. 

Arguments: 'tasks.simple_task' and input form parameters are passed to **__add_task_to_queue**. Good practice to pass string pointers to worker functions rather than passing function instances (python tries to pickle them to pass to workers, which it doesn't always succeed), so you'd better separate application code and background tasks code, passed to worker.
### tasks.simple_task
```python
from time import sleep
def simple_task(task_arg):
	sleep(60)
return "prefix___"+task_arg
```
A "toy" job, all the function has to do is wait 60 seconds (to populate our queue) and argument (task_arg) with prefix.
##  Closing thoughts
RQ has its [dashboard monitor](https://python-rq.org/docs/monitoring/) for monitoring queues.
It can be add as additional Docker service.
**local_image/flask_rq** can be splitted to image for app and worker.The same applies to the code(mounting **src** folder to app and **tasks** folder to worker)

RQ has its own [dashboard monitor](https://python-rq.org/docs/monitoring/)  for monitoring queues.
It can be added as an additional Docker service.
**local_image/flask_rq** can be split into images per app and worker. The same goes for the code (mounting the **src** folder for the app and the **tasks** folder for the worker).

 

