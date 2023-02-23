# Asynchronous Tasks with Flask,RQ and Redis in Docker
This tutorial will walk you through deploying up a basic Flask app using background jobs with the help of Docker containers. Code template can be downloaded from [github/Rayveni](https://github.com/Rayveni/blog/tree/main/flask%20redis)
## Table of Contents
A Docker container is a sealed environment where you can run your application and all its dependencies. The image contains all the tools and libraries needed to run your application, including any necessary setup files.  
Docker images are created from a set of files that describe how to run the application containers in production and development.

Flask is a lightweight Python-based micro web framework, which it easy to build web applications and REST APIs with its built-in HTTP request/response system.

Redis is an open source in-memory **data structure store** used as a database, cache, message broker, and streaming engine.

RQ is a powerful library for managing asynchronous task queues and workflows on top of Redis.
## Prerequisite
-   Install Docker Desktop
Visit [https://www.docker.com/get-started/](https://www.docker.com/get-started/) to download Docker Desktop for your system.
## Launching the application
Perform next steps:

 1. Download example code  from [github/Rayveni](https://github.com/Rayveni/blog/tree/main/flask%20redis)
 2. Navigate to project folder in terminal and execute:
 ![enter image description here](https://raw.githubusercontent.com/Rayveni/blog/main/articles/flask%20redis/img/terminal-min.png)
     ```bash
     docker-compose --env-file .env build
     ```
     This command creates docker images for application.
 4. To run container with application built from previous step images :
	   ```bash
		 docker-compose --env-file .env up -d
	```
 5.  In browser navigate to **localhost:5000**:
![enter image description here](https://raw.githubusercontent.com/Rayveni/blog/main/articles/flask%20redis/img/app_screen.jpg)
 7. Run some background jobs by submitting task params
 in input form(in provided example background task is will return input param with 60seconds delay) and check results (click link *Completed tasks*).
Explanations of project code  below.
 9.  To stop container and app:	  
	   ```bash
	   docker-compose --env-file .env down
		``` 
 
 ## Project Structure 
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

In this folder structure, the **backend** folder contains :
 1.   **src** folder— flask application location and corresponding APIs.
 2.   **tasks** folder— This is where code for async jobs stored
 3.   server.py — http server launcher (I use gevent server )
 4.   worker.py —  redis queue listener and job executor
 5.   **Dockerfile** —  is a text file that contains all the commands that compose a Docker image(to make it easy i did one image to app and worker).
## Docker-compose
In paragraph   **Launching the application** parameter **env-file** was applied to docker-compose command.

**docker-compose** supports parameter expansion in environment files. In simple words (`${..}`)  replaced when running docker-compose with corresponding named params(in our project this params stored in **.env** file).
This approach allows easily configure environment, container run in (e.g. prod/dev/test) .
<script src="https://gist.github.com/Rayveni/9f55f67af6dfe4442f228e9213dcea4e.js"></script>
 following images created:
 

 - local_image/flask_rq - from **Dockerfile** in backend folder for our flask application
 - image for redis db -from official docker hub

**Dockerfile** is pretty straight forward(simple installation necessary python libs from requirements file):
```bash
FROM python:3.9.16-slim-bullseye
COPY requirements.txt /
RUN pip3 install --no-cache-dir -r /requirements.txt
WORKDIR /app
```
 The use of the same image for  redis queue listener and flask  application is a little bit tricky, but it was done for simplicity. Worker(queue lister) and app server  distinguished by **command:python worker.py** and **command:server.py** in docker-compose.yml.

There is 2 options to place code in Docker:

 - copy into image with help of Dockerfile,using copy command(same as *COPY requirements.txt* ) which is better way for production,but requires time to rebuild image if app code changes
 - mount code as folder from host -better way for application development ,because changes applied instantly

In this project   **backend** folder  mounted for worker and app services. Keep in mind that in real project  worker code and app code should be independent.
## worker and server
<script src="https://gist.github.com/Rayveni/44690444d49bf881f0f658c94f072e52.js"></script>

 - **server.py** -launches gevent http server on docker virtual network ("0.0.0.0:80").
 To make it visible on host we use in docker-compose(after replacement from .env)
	 ```bash
	 ports:
	- "5000:80"
	 ```
	 80-port for docker virtual network(you cound use anyone) "0.0.0.0:**80**"+"5000:**80**"->localhost:5000
 - **worker.py**- launches listener to redis queue (redis://redis:6379).There could be multiple queues (for e.g. queue for high /low/medium priority could be designed).In our project used only one queue -**default**.
## Flask Application
<script src="https://gist.github.com/Rayveni/b3fc00fe9dd068dbae055289e0c21a49.js"></script>
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
 - "/"  -for index page (rendering **index.html** from templates folder)
 - finished_tasks-  json with finished tasks info

Post request from the **index.html**  calls **__add_task_to_queue** function, which simply adds tasks to **default** queue(as we remember, launched worker listens this queue).

Arguments:  'tasks.simple_task' and input form parameter are passed to **are__add_task_to_queue**.Good practice to pass string pointers to worker function instead passing instances of functions (python will try to pickle them to pass to worker and not always succeed in it),so better writhe independent from each other app and worker code. 
### tasks.simple_task
```python
from time import sleep
def simple_task(task_arg):
	sleep(60)
return "prefix___"+task_arg
```
A "toy" task, all we do is waiting 60 seconds(to fill our queue) and returning argument with prefix.

##  What is Next?
RQ has its [dashboard monitor](https://python-rq.org/docs/monitoring/) for monitoring queues.
It can be add as additional Docker service.
**local_image/flask_rq** can be splitted to image for app and worker.The same applies to the code(mounting **src** folder to app and **tasks** folder to worker)

 

