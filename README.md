# Docker

# A platform for Building, Running & Shipping Applications. 
  If an application runs on developer machine it can run the same way on other machines.
  Situation arise where an application works on developer machine but not on remote machine, this is were the Docker comes for rescue.

## Architecture:
	It uses CLIENT-SERVER model. Communication happens via REST-API.
	SERVER is also known as DOCKER ENGINER. It sits in the background and build & run a Docker Containers.
	Typically a CONTAINER is like a Process similar to other processes running on our system/computer but a special kind which contains its own file system(virtual). Container is a running environment for image.
	On a HOST System multiple CONTIANERS can run, all sharing the OS of the HOST system itself making it lightweight. 
	
## Dockerfile:
	A Dockerfile is a plain text file which includes instructions which docker uses to package the application into an IMAGE.
	This DOCKER IMAGE contains everything an application needs to run.
	The IMAGE is used by the DOCKER to START a CONTAINER.
	The IMAGE can be pushed to DOCKER-REGISTRY/DOCKER-HUB. This is similar to GIT/GIT-HUB where we are store this images and anyone can use it.
	
	->Dockerfile basic Instructions for nodeJs code:
		FROM node:alpine
		COPY . /app
		WORKDIR /app
		RUN npm install
		CMD ["node", "app.js"]
		
	->DOCKER COMMANDS:
		=> command packages the application into an image with a tag name in current directory
			docker build -t hello-docker .
		=> command to create a new TAG for the image before to be pushed to PRIVATE REPOSITRY of a REGISTRY
			docker tag hello-docker:latest registry-host:5000/myadmin/hello-docker:1.0
		=> Command to check available images in the system
			docker images 
			docker image ls
		=> Command to RUN the image
			docker run hello-docker
		=> Command to push the image to REGISTRY
			docker image push registry-host:5000/myadmin/hello-docker:1.0
			docker push registry-host:5000/myadmin/hello-docker:1.0
		=> Command to pull the image in another system/container
			docker pull codewithmosh/hello-docker
		=> Command to check running Docker processes/Containers
			docker ps
		=> Command to check stopped Docker processes/Containers
			docker ps -a
		=> Command to RUN a CONTAINER in INTERACTIVE mode
			docker run -it ubuntu
		=> Command to EXECUTE a RUNNING CONTAINER in INTERACTIVE mode using CONATINER-ID/NAME & go inside Container & open SHELL
			docker exec -it {CONTAINER-ID}/{CONTAINER-NAME} /bin/bash
		=> Command to RUN a CONTAINER in DETACH mode
			docker run -d nginx:1.23
		=> Command to RUN a CONTAINER in DETACH mode with PORT-BINDING
			(Contianers run in Isolated env. To expose them, we use PORT-BINDING. Port-9000 is HOST port whereas Port-80 is of CONTAINER)
			docker run -d -p 9000:80 nginx:1.23
		=> Command to START/STOP an exiting Contianer
			docker start {CONTAINER-ID}
			docker stop {CONTAINER-ID}
		=> Command to RUN a CONTAINER and give it a NAME
			docker run --name web-app nginx:1.23
		=> COMMAND to check logs
			docker logs {CONTAINER-ID}
		=> COMMAND to Delete a CONTAINER
			docker rm {CONTAINER-ID}
		=> COMMAND to ALL CONTAINERS
			docker container prune
		=> Command to find IP-ADDRESS of Running CONTAINER
			docker inspect {CONTAINER-ID} | findstr IPAddress
	
## REGISTRY vs REPOSITORY:
		REGISTRY-> A service poroviding storage. It is a collection of REPOSITORY.
		REPOSITORY-> Collection of related IMAGES with same NAME but diff VERSIONS.

## NETWORK:
	Docker provides its own network and we can also create a network/connection between different containers.
	
	=> Command to Check NETWORK provided by DOCKER
			docker network ls
	=> Command to CREATE NETWORK
			docker network create mongo-network
			
## CONENCTING TO MONGODB & MONGO-EXPRESS USING CUSTOM-NETWORK:
	=>While developing a code we would like to connect both MONGODB and MONGO-EXPRESS Container to connect with each other via 	a CUSTOM-NETWORK create above earlier along with PORT-BINDING.
	=> Our APLLICATION CODE present outside container can connect to MONGO DB via URL containing credential & HOST-port number(27017)
	
	=> Command to Run a MONGODB docker container within our custom network, custom container name, setting up Env-variable 
			docker run -d \
			-p 27017:27017 \
			-e MONGO_INITDB_ROOT_USERNAME=admin \
			-e MONGO_INITDB_ROOT_PASSWORD=password \
			--name mongodb \
			--network mongo-network \
			mongo
	=> Command to Run a MONGO-EXPRESS docker container within our custom network, custom container name, setting up Env-variable
		ME_CONFIG_MONGODB_SERVER ENV-VAR uses the MONGODB CONTAINER-NAME
			docker run -d \
			-p 8080:8081
			-e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
			-e ME_CONFIG_MONGODB_ADMINPASSWORD==password \
			--name mongo-express \
			--network mongo-network \
			-e ME_CONFIG_MONGODB_SERVER=mongodb \
			mongo-express

## DOCKER-COMPOSE:
	=>Defining and running multi-container Docker applications.
	=>While working with multiple CONTAINERS required by our application, we dont want to RUN this containers manually.
	=>We can use docker-compose yaml file to mention all the container required to be started and communicate with each other in a   	structured manner.
	=>We are not required to create a CUSTOM COMMON NETWORK while using docker comopose. docker-compose creates and deletes the common network while executing and stopping the docker-cponse file.
	
	Below is an example of how the docker compose yaml file will look like.
		
		version: '3'
		services:
			my-app:
				image: registry-host:5000/myadmin/hello-docker:latest
				ports:
				 - 3000:3000
			mongodb:
				image: mongo
				ports:
				 - 27017:27017
				environments:
				 - MONGO_INITDB_ROOT_USERNAME=admin
				 - MONGO_INITDB_ROOT_PASSWORD=password
			mongo-express:
				image: mongo-express
				ports:
				 - 8080:8081
				environments:
				 - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
				 - ME_CONFIG_MONGODB_ADMINUPASSWORD=password
				 - ME_CONFIG_MONGODB_SERVER=mongodb
	
	=> COMMAND to RUN DOCKER-COMPOSE YAML file:
		docker-compose -f filename.yaml up
	=> COMMAND to STOP all containers using DOCKER-COMPOSE YAML file:
		docker-compose -f filename.yaml down

## DOCKER-VOLUMES:
		=> Directory/Folder in physical HOST file system is MOUNTED  into the VIRTUAL file sysem of DOCKER.
		=> When a data is written on the Virtual file system it is automatically replicated onto the HOST system. This helps in persistance data storage when the container is getting restarted it will retain the data.
		=> Used for databases and stateful applications.
	
		##3-VOLUME TYPES:
				1-> HOST-VOLUME:
				        docker run -v HOST-FILE-SYSTEM:VIRTUAL-FILE-SYSTEM
						docker run -v /home/mount/data:/var/lib/mysql/data
						
					A user can decide where on the HOST file system the data is required to be mounted/replicated
				
				2-> ANONYMOUS-VOLUME:
						docker run -v :VIRTUAL-FILE-SYSTEM
						docker run -v /var/lib/mysql/data
						
					The HOST file system is not mentioned. For each CONTAINER a folder is generated that gets mounted automatically.
					This volumes are known as ANONYMOUS VOLUMES as we dont the the reference of the generated folder.
				
				3-> NAMED-VOLUME:
						docker run -v name:VIRTUAL-FILE-SYSTEM
						docker run -v db-data:/var/lib/mysql/data
						
					User can reference the volume by NAME. This is most widely used volume for PROD-ENV.
					Same volume by using named reference can be used for MULTIPLE-CONTAINERS.
				
	Below is an example of how the docker compose yaml file will look with DOCKER-VOLUME
		
			version: '3'
			services:
				my-app:
					image: registry-host:5000/myadmin/hello-docker:latest
					ports:
					 - 3000:3000
				mongodb:
					image: mongo
					ports:
					 - 27017:27017
					environments:
					 - MONGO_INITDB_ROOT_USERNAME=admin
					 - MONGO_INITDB_ROOT_PASSWORD=password
					volumes:
					 - db-data:/var/lib/mysql/data
				mongo-express:
					image: mongo-express
					...
			volumes:
				db-data:
					driver: local
					
## Command to connect to DOCKER POSTGRES-DB CONTAINER with HOST system PgAdmin:
	docker run --name postgres-db -p 5432:5432 -e POSTGRES_PASSWORD=MauryaPg25* -d postgres:16.1-alpine3.18
