### Python/Flask application using a Redis database
 
### Project structure:
```
.
├── Dockerfile
├── README.md
├── app.py
├── docker-compose.yaml
└── requirements.txt
```

### Create a directory for the project:
```
$ mkdir flask-redis
$ cd flask-redis
```

### Create a file called app.py in your project directory and paste the following code in:
```
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```
In this example, redis is the hostname of the redis container on the application’s network. We use the default port for Redis, 6379.

### Create another file called requirements.txt in your project directory and paste the following code in:
```
flask
redis
```

### Create a Dockerfile
The Dockerfile is used to build a Docker image. The image contains all the dependencies the Python application requires, including Python itself.

In your project directory, create a file named Dockerfile and paste the following code in:
```
# syntax=docker/dockerfile:1
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```
This tells Docker to:

Build an image starting with the Python 3.7 image.
Set the working directory to /code.
Set environment variables used by the flask command.
Install gcc and other dependencies
Copy requirements.txt and install the Python dependencies.
Add metadata to the image to describe that the container is listening on port 5000
Copy the current directory . in the project to the workdir . in the image.
Set the default command for the container to flask run.

### Define services in a Compose file

Create a file called docker-compose.yml in your project directory and paste the following:
```
version: "3.9"
services:
   redis:
     image: "redis:alpine"
     ports:
       - '6379:6379'
   web:
     build: .
     ports:
       - "8000:5000"
     volumes:
       - .:/code
     environment:
       FLASK_DEBUG: "true"
     depends_on:
       - redis
```
This Compose file defines two services: web and redis.

The web service uses an image that’s built from the Dockerfile in the current directory. It then binds the container and the host machine to the exposed port, 8000. This example service uses the default port for the Flask web server, 5000.

The redis service uses a public Redis image pulled from the Docker Hub registry.

### Build and run your app with Compose

From your project directory, start up your application by running docker compose up
```
$ docker-compose up -d
Pulling redis (redis:alpine)...
alpine: Pulling from library/redis
8a49fdb3b6a5: Pull complete
2121af5b35c8: Pull complete
13ce36f68149: Pull complete
d0786890719f: Pull complete
e0001f8e7e31: Pull complete
4ea079d1c234: Pull complete
Digest: sha256:1f27b9eb680ffcf6c68966c0d5f578bb1b030ca7cd8ec4e758c429e7f72005a0
Status: Downloaded newer image for redis:alpine
Building web
Sending build context to Docker daemon  72.19kB
Step 1/10 : FROM python:3.7-alpine
3.7-alpine: Pulling from library/python
8a49fdb3b6a5: Already exists
0357922e53aa: Pull complete
db547a433dc2: Pull complete
62dd39903263: Pull complete
eb20fb6b291e: Pull complete
Digest: sha256:f48c5f6a8a22a73558ea93eb26d2c7928d23f2acb2bb9270be9a08adc2bfa63d
Status: Downloaded newer image for python:3.7-alpine
 ---> e4fbc12a05a9
Step 2/10 : WORKDIR /code
 ---> Running in 08e2a7f9e5f8
Removing intermediate container 08e2a7f9e5f8
 ---> 8acc59860a9a
Step 3/10 : ENV FLASK_APP=app.py
 ---> Running in 09e45ca1bfe9
Removing intermediate container 09e45ca1bfe9
 ---> 3763bc76581d
Step 4/10 : ENV FLASK_RUN_HOST=0.0.0.0
 ---> Running in efa1e8070f98
Removing intermediate container efa1e8070f98
 ---> 05cd4e0c23b8
Step 5/10 : RUN apk add --no-cache gcc musl-dev linux-headers
 ---> Running in f0108d831829
fetch https://dl-cdn.alpinelinux.org/alpine/v3.18/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.18/community/x86_64/APKINDEX.tar.gz
(1/13) Installing libgcc (12.2.1_git20220924-r10)
(2/13) Installing libstdc++ (12.2.1_git20220924-r10)
(3/13) Installing zstd-libs (1.5.5-r4)
(4/13) Installing binutils (2.40-r6)
(5/13) Installing libgomp (12.2.1_git20220924-r10)
(6/13) Installing libatomic (12.2.1_git20220924-r10)
(7/13) Installing gmp (6.2.1-r3)
(8/13) Installing isl26 (0.26-r1)
(9/13) Installing mpfr4 (4.2.0-r2)
(10/13) Installing mpc1 (1.3.1-r1)
(11/13) Installing gcc (12.2.1_git20220924-r10)
(12/13) Installing linux-headers (6.3-r0)
(13/13) Installing musl-dev (1.2.4-r0)
Executing busybox-1.36.0-r9.trigger
OK: 166 MiB in 51 packages
Removing intermediate container f0108d831829
 ---> f7531097573d
Step 6/10 : COPY requirements.txt requirements.txt
 ---> 1ec95f67a622
Step 7/10 : RUN pip install -r requirements.txt
 ---> Running in 36ccddf9997d
Collecting flask
  Downloading Flask-2.2.5-py3-none-any.whl (101 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 101.8/101.8 KB 6.1 MB/s eta 0:00:00
Collecting redis
  Downloading redis-4.5.5-py3-none-any.whl (240 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 240.3/240.3 KB 28.3 MB/s eta 0:00:00
Collecting Jinja2>=3.0
  Downloading Jinja2-3.1.2-py3-none-any.whl (133 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 133.1/133.1 KB 20.1 MB/s eta 0:00:00
Collecting importlib-metadata>=3.6.0
  Downloading importlib_metadata-6.6.0-py3-none-any.whl (22 kB)
Collecting Werkzeug>=2.2.2
  Downloading Werkzeug-2.2.3-py3-none-any.whl (233 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 233.6/233.6 KB 31.2 MB/s eta 0:00:00
Collecting itsdangerous>=2.0
  Downloading itsdangerous-2.1.2-py3-none-any.whl (15 kB)
Collecting click>=8.0
  Downloading click-8.1.3-py3-none-any.whl (96 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 96.6/96.6 KB 15.5 MB/s eta 0:00:00
Collecting async-timeout>=4.0.2
  Downloading async_timeout-4.0.2-py3-none-any.whl (5.8 kB)
Collecting typing-extensions
  Downloading typing_extensions-4.5.0-py3-none-any.whl (27 kB)
Collecting zipp>=0.5
  Downloading zipp-3.15.0-py3-none-any.whl (6.8 kB)
Collecting MarkupSafe>=2.0
  Downloading MarkupSafe-2.1.2-cp37-cp37m-musllinux_1_1_x86_64.whl (30 kB)
Installing collected packages: zipp, typing-extensions, MarkupSafe, itsdangerous, Werkzeug, Jinja2, importlib-metadata, async-timeout, redis, click, flask
Successfully installed Jinja2-3.1.2 MarkupSafe-2.1.2 Werkzeug-2.2.3 async-timeout-4.0.2 click-8.1.3 flask-2.2.5 importlib-metadata-6.6.0 itsdangerous-2.1.2 redis-4.5.5 typing-extensions-4.5.0 zipp-3.15.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
WARNING: You are using pip version 22.0.4; however, version 23.1.2 is available.
You should consider upgrading via the '/usr/local/bin/python -m pip install --upgrade pip' command.
Removing intermediate container 36ccddf9997d
 ---> e9d0b4a766c9
Step 8/10 : EXPOSE 5000
 ---> Running in e02c46f6716f
Removing intermediate container e02c46f6716f
 ---> 75484c272a2a
Step 9/10 : COPY . .
 ---> b913476af79a
Step 10/10 : CMD ["flask", "run"]
 ---> Running in f6055a332e4f
Removing intermediate container f6055a332e4f
 ---> 443f8f92a1e8
Successfully built 443f8f92a1e8
Successfully tagged flask-redis_web:latest
WARNING: Image for service web was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Creating flask-redis_redis_1 ... done
Creating flask-redis_web_1   ... done
```
Compose pulls a Redis image, builds an image for your code, and starts the services you defined. In this case, the code is statically copied into the image at build time.

Enter http://localhost:8000/ in a browser to see the application running.

If this doesn’t resolve, you can also try http://127.0.0.1:8000.

You should see a message in your browser saying:

+ Hello from Docker! I have been seen 1 times.

Refresh the page.

The number should increment.

+ Hello from Docker! I have been seen 2 times.

### Switch to another terminal window, and type docker image ls to list local images.

Listing images at this point should return redis and web.
```
ubuntu@ip-172-31-24-87:~/flask-redis$ docker images
REPOSITORY        TAG          IMAGE ID       CREATED          SIZE
flask-redis_web   latest       443f8f92a1e8   16 minutes ago   214MB
redis             alpine       f37f9f678836   8 days ago       30.2MB
python            3.7-alpine   e4fbc12a05a9   9 days ago       47MB
```

### You can inspect images with docker inspect <tag or id>.

Stop the application, either by running docker compose down from within your project directory in the second terminal, or by hitting CTRL+C in the original terminal where you started the app.
 
### Update the application

Because the application code is now mounted into the container using a volume, you can make changes to its code and see the changes instantly, without having to rebuild the image.

Change the greeting in app.py and save it. 
 
For example, change the Hello World! message to Hello from Docker!:
```
return 'Hello from Docker! I have been seen {} times.\n'.format(count)
```

Refresh the app in your browser. The greeting should be updated, and the counter should still be incrementing.

### Connect to redis database by using ```redis-cli``` command and monitor the keys. 
```
redis-cli -p 6379
127.0.0.1:6379> monitor
OK
1646634062.732496 [0 172.21.0.3:33106] "INCRBY" "hits" "1"
1646634062.735669 [0 172.21.0.3:33106] "GET" "hits"
```

### Stop and remove the containers and volumes
```
$ docker compose down --volumes
```
