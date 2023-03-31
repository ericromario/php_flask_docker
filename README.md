# student-list 
This repo is a simple application to list student with a webserver (PHP) and API (Flask)

![project](https://user-images.githubusercontent.com/18481009/84582395-ba230b00-adeb-11ea-9453-22ed1be7e268.jpg)


------------


## Context

*POZOS* is an IT company located in France that develops software for secondary schools.

The innovation department wanted to overhaul the existing infrastructure to ensure that

that it can be scalable, easily deployed with maximum automation.

POZOS wants me to do a **POC** to show how docker can help and how efficient the technology is.

For this POC, POZOS gives me an application and asks me to build a "decoupled" infrastructure based on **Docker**.

Currently, the application runs on a single server without any scalability or high availability.

When POZOS needs to deploy a new version, there is always a problem.

In conclusion, POZOS needs agility on its software farm.

### Tasks:

- Containerise the Flask application 
- Build run and test the application with a simple curl
- Create a docker compose file to automate the deployment of the application and its GUI using the php:apache image
- Test the application again via its GUI
- create the release 
- Deploy the private registry and its GUI
- Push the realese and visualize it


## Infrastructure

For this POC, I will only use one single machine with a docker installed on it.

The build and the deployment will be made on this machine.

POZOS recommends me to use centos7.6 OS because it's the most used in the company.

POZOS requires me to use a virtual machine based on Centos7.6 and not my physical machine.
## Application

The application that I will be working on is named "*student_list*", this application is very basic and enables POZOS to show the list of the student with their age.

student_list has two modules:

- the first module is a REST API (with basic authentication needed) who send the desire list of the student based on JSON file
- The second module is a web app written in HTML + PHP who enable end-user to get a list of students

I work is to build one container for each module an make them interact with each other

### file's role:

- docker-compose.yml: to launch the application (API and web app)
- Dockerfile: the file that will be used to build the API image (details will be given)
- student_age.json: contain student name with age on JSON format
- student_age.py: contains the source code of the API in python
- index.php: PHP  page where end-user will be connected to interact with the service to - list students with age. You need to update the following line before running the website container to make ***api_ip_or_name*** and ***port*** fit your deployment
   ` $url = 'http://<api_ip_or_name:port>/pozos/api/v1.0/get_student_ages';`



## Build and test 

POZOS will give me information to build the API container

- Base image

To build API image I must use "python:2.7-stretch"

- Add the source code

I need to copy the source code of the API in the container at the root "/" path

- Prerequisite

The API is using FLASK engine,  here is a list of the package I need to install
```
apt-get update -y && apt-get install python-dev python3-dev libsasl2-dev python-dev libldap2-dev libssl-dev -y
pip install flask==1.1.2 flask_httpauth==4.1.0 flask_simpleldap python-dotenv==0.14.0
```
- Persistent data (volume)

I create data folder at the root "/" where data will be stored and declare it as a volume

You will use this folder to mount student list

- API Port

To interact with this API expose 5000 port

- CMD

When container start, it must run the student_age.py (copied at step 4), so it should be something like

`CMD [ "python", "./student_age.py" ]`

Build your image and try to run it (don't forget to mount *student_age.json* file at */data/student_age.json* in the container), check logs and verify that the container is listening and is  ready to answer

Run this command to make sure that the API correctly responding (take a screenshot for delivery purpose)

`curl -u toto:python -X GET http://<host IP>:<API exposed port>/pozos/api/v1.0/get_student_ages`

## Infrastructure As Code 

After testing your API image, you need to put all together and deploy it, using docker-compose.

The ***docker-compose.yml*** file will deploy two services :

- website: the end-user interface with the following characteristics
   - image: php:apache
   - environment: you will provide the USERNAME and PASSWORD to enable the web app to access the API through authentication
   - volumes: to avoid php:apache image run with the default website, we will bind the website given by POZOS to use. You must have something like
`./website:/var/www/html`
   - depend on: you need to make sure that the API will start first before the website
   - port: do not forget to expose the port
- API: the image builded before should be used with the following specification
   - image: the name of the image builded previously
   - volumes: You will mount student_age.json file in /data/student_age.json
   - port: don't forget to expose the port

Delete your previous created container

Run your docker-compose.yml

Finally, reach your website and click on the bouton "List Student"

## Docker Registry 

POZOS need I to deploy a private registry and store the built images

So I need to deploy :

- a docker [registry](https://docs.docker.com/registry/ "registry")
- a web [interface](https://hub.docker.com/r/joxit/docker-registry-ui/ "interface") to see the pushed image as a container

I push image on the private registry and show them in your delivery.

![good luck](https://user-images.githubusercontent.com/18481009/84582398-cad38100-adeb-11ea-95e3-2a9d4c0d5437.gif)
