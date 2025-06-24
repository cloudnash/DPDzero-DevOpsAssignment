### 🧪 **DevOps Intern Assignment: Nginx Reverse Proxy + Docker**


1. First I ran ec2 machine on AWS Ubuntu OS.

2. I tried to get the same folders to a remote repo so that we can pull service-1 and service-2 to Ubuntu machine to make the required directory setup

### 📁 mkdir on project structure

```
├── service_1
│   ├── app.py
├── service_2
│   ├── app.py
└── README.md
```

Screenshot is also attached and can refer to service-1 and service-2 JPGs.


3. Now that we have the 2 folder with README.md , I am going to test it locally first run the Golan and Python on their ports to see and confirm the endpoints.

	## SERVICE-1 TESTING LOCALLY

	a. While running the service-1 which has Golang we need to have prerequisite 'go' installed on the machine and then we run

	### Installing go on machine:

	$ wget https://go.dev/dl/go1.22.3.linux-amd64.tar.gz
	$ sudo tar -C /usr/local -xzf go1.22.3.linux-amd64.tar.gz
	$ nano ~/.bashrc
	$ export PATH=$PATH:/usr/local/go/bin
	$ source ~/.bashrc
	$ go --version

	$ go mod init service1

	
	#### Run with Go:

	```bash
	go run main.go


	```
so after running this I checked my Then visit:

* [http://localhost:8001/ping](http://localhost:8001/ping)
* [http://localhost:8001/hello](http://localhost:8001/hello).

NOTE: I had allowed all the Inbound traffic from 8001 in our security group of EC2 machine.


Only in my case I am running this is EC2 machine of PUBLIC IP: http://3.80.211.14 that can listens to port 8001 which you can see in see in the screenshot  go-A and go-B





4. Same I got for service-2 python api service.

	## SERVICE-2 TESTING LOCALLY

	a. While running the service-2 which has Golang we need to have prerequisite python and uv both installed on the machine as per the video suggested and can refer from service-2 README.md

	### installing python and uv

	$ sudo apt update
	$ sudo apt install python3 python3-pip -y
	$ sudo apt install python3-venv -y      (for Isolated environment)
	$ python3 -m venv venv
	$ source venv/bin/activate

	$ wget -qO- https://astral.sh/uv/install.sh | sh   ##instaling uv 
	$ pip install uv

	#### Run with uv:

	```bash
	uv run app.py

	```
so after running this I checked my Then visit:

* [http://localhost:8002/ping](http://localhost:8002/ping)
* [http://localhost:8002/hello](http://localhost:8002/hello).

NOTE: I had allowed all the Inbound traffic from 8002 in our security group of EC2 machine.


Only in my case I am running this is EC2 machine of PUBLIC IP: http://3.80.211.14 that can listens to port 8002 which you can see in see in the screenshot  py-A and py-B



5. Now, according to the I am going to dockerize them on each services after installing the docker



	## installing docker
	
	$ sudo apt update
	$ sudo apt-get install docker.io
	$ sudo usermod -aG docker ubuntu


	## Dockerizing the service-1
	
	FROM golang:1.21-alpine

	WORKDIR /app

	COPY . .

	RUN go build -o main .

	CMD ["./main"]



	## Dockerizing the service-2

	FROM python:3.12-slim

	WORKDIR /app

	COPY . .

	RUN pip install fastapi uvicorn

	CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8002"]



Our project directory looks like this now reference to screenshot service-1-docker and service-2-docker



### 📁 currently project structure

```
.
├── service_1
│   ├── app.py
│   └── Dockerfile
├── service_2
│   ├── app.py
│   └── Dockerfile
└── README.md




6. After dockerizing the services I am going for nginx reverse proxy and configuration in a such way it talk to both service-1 and service-2 with containerizing the nginx itself. so that It can work with docker-compose


	### we are setting up "nginx.config" for it communicate and receive and send traffic/ messages from service-1 and services-2 to localhost.

	
		1. **Nginx reverse proxy** (also in a Docker container) routes:
   			* `/service1` requests to backend service 1
   			* `/service2` requests to backend service 2
		2. All services must be accessible via a single port (e.g., `localhost:8080`)


	## default.config

events {}

http {
    log_format custom '[$time_local] "$request_uri"';
    access_log /var/log/nginx/access.log custom;

    server {
        listen 80;

        location /service1/ {
            proxy_pass http://service1:8001/;
        }

        location /service2/ {
            proxy_pass http://service2:8002/;
        }
    }
}



	### And then Dcokerizing the default.conf

	FROM nginx:latest
	COPY default.conf /etc/nginx/nginx.conf


now 📁 directory looks like this

```
├── nginx
│   ├── default.conf
│   └── Dockerfile
├── service_1
│   ├── app.py
│   └── Dockerfile
├── service_2
│   ├── app.py
│   └── Dockerfile
└── README.md

```





7. Moving to the next, since we have connected and dockerized everything, we are going to be orchestrating the whole docker system with docker compose and we are going to create its YAML file.

Provided the docker-compose is installed in the machine.

	### Installing docker-compose

	$ sudo apt-get update
	$ sudo apt-get install docker-compose-plugin
		


	### creating docker-compose.yaml


version: '3.8'

services:
  service1:
    build: ./service_1
    container_name: service1

  service2:
    build: ./service_2
    container_name: service2

  nginx:
    build: ./nginx
    ports:
      - "8080:80"
    depends_on:
      - service1
      - service2


Now the whole directory looks like desired project directory and we can confirm from screenshot desirect-directory.



.
├── docker-compose.yml
├── nginx
│   ├── default.conf
│   └── Dockerfile
├── service_1
│   ├── app.py
│   └── Dockerfile
├── service_2
│   ├── app.py
│   └── Dockerfile
└── README.md


and finally we are going to build it the command

''''

docker compose up --build

''''



***ERROR 1 : we have encountered and error while perform docker-compose 

	Apparently it says "ModuleNotFoundError: No module named 'flask' "


	We are going to make some changes in app.py, which is

	### Updating service_2 Dockerfile


	FROM python:3.12-slim

	WORKDIR /app

	COPY requirements.txt .

	RUN pip install -r requirements.txt

	COPY . .

	CMD ["python", "app.py"]




***ERROR 2 : we have encountered another error in nginx.conf, so we are going to reconfigure the deafualt.conf and Dockerfile


-----defualt.conf------


server {
    listen 80;

    location /service1/ {
        proxy_pass http://service1:8001/;
    }

    location /service2/ {
        proxy_pass http://service2:8002/;
    }
}



------Dockerfile------


FROM nginx:latest
COPY default.conf /etc/nginx/conf.d/default.conf








OUTPUT:


The Docker-compose was build the whoel setup and we were abel to see the following output


http://3.80.211.14:8080/service1/ping   -    Pretty-print
							{"service":"1", "status":"ok"}

http://3.80.211.14:8080/service2/ping   -    Pretty-print
							{"service":"2", "status":"ok"}







And Final Push to remote Repo


cd ~/DPDzero-DevOpsAssignment

git init

git add .
git commit -m "Initial commit: DPDzero DevOps Assignment"


git remote add origin https://github.com/cloudnash/DPDzero-DevOpsAssignment.git

git branch -M main
git push -u origin main --force



*****Is my remote GitHub repo:   https://github.com/cloudnash/DPDzero-DevOpsAssignment




Made by Nashit Ahmad
