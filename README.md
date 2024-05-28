<div style="text-align: center;">
    <img src="https://www.effseit.fr/public/img/medium/logoepfpng_642580bd3298c0.78845696.png" alt="EPF Logo" width="200">
</div>

# DevOps - Practical Works 
> Théo BURDINAT - EPF MDE P2025

## Table of contents

* [PW n°1](#pw-n1)
    * [Question 1-1](#1-1-document-your-database-container-essentials-commands-and-dockerfile)
    * [Question 1-2](#1-2-why-do-we-need-a-multistage-build-and-explain-each-step-of-this-dockerfile)
    * [Question 1-3](#1-3-document-docker-compose-most-important-commands)
    * [Question 1-4](#1-4-document-your-docker-compose-file)
    * [Question 1-5](#1-5-document-your-publication-commands-and-published-images-in-dockerhub)

## PW n°1

### 1-1. Document your database container essentials: commands and Dockerfile.

To deploy the database, I ran the following commands :

* Create the Docker image : (from the *postgres* folder)

    `docker build -t theoburdinat/mydatabase .`

    It will create an image from the database Dockerfile, named theoburdinat/mydatabase.

* Create the network that will permit to communicate between our containers :

    `docker network create app-network`

* Deploy our database container :

    `docker run -p 5432:5432 --name myPostgres -e POSTGRES_USER=theo -e POSTGRES_PASSWORD='pa$$' --network app-network -v '"C:\Users\theob\OneDrive - Fondation EPF\4A\S8\Devops\tp1\postgres\data":/var/lib/postgresql/data' -d theoburdinat/mydatabase`

    This command permits to deploy a container from the image created fromm our Dockerfile. We used environment variables to avoid having our credentials in clear text in a file, which isn't exactly top-notch in terms of security. We also connected this container to our network, and created a volume that will permits to save our data, even if our container gets destroyed.

* Deploy an *adminer* container :

    `docker run -p 8090:8080 --network app-network --name adminer -d adminer`

    This application permits to manage our database. It is accessible from the port 8090 and we can connect to our database from http://localhost:8090/ with our credentials.

Now, let's check our Dockerfile :

```dockerfile
FROM postgres:14.1-alpine # We decided the official PostgreSQL image, version 14.1, built on the lightweight Alpine Linux distribution. This is beneficial for keeping the image size small and for faster deployment.

ENV POSTGRES_DB=db # Specification of the default database's name

COPY ./docker-entrypoint-initdb.d/ /docker-entrypoint-initdb.d/ # Takes all the file from the local directory and copy them to the container's directory. This PostgreSQL special directory will permits to run all the SQL files during the initialization phase of the container. Here it is very useful to set up the database schema and preload some data.
```

### 1-2. Why do we need a multistage build? And explain each step of this dockerfile.

```dockerfile
# Build
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build # defines the base image for the build stage
# We use the Maven image with Amazon Corretto 17, which is a distribution of the OpenJDK. This image includes all the tools needed to build a Java application using Maven.
ENV MYAPP_HOME /opt/myapp 
WORKDIR $MYAPP_HOME # set up the working directory for the build (where all subsequent commands will be run)
COPY pom.xml . # Maven configuration file
COPY src ./src # Folder that contains the Java source code
RUN mvn package -DskipTests # Compile the code and package it into a JAR file
RUN mvn dependency:go-offline # Download all the dependencies required for the project and store them locally, to avoid having to download all the librairies on each image build.

# Run
FROM amazoncorretto:17 # defines the base image for the run stage
# We use the Amazon Corretto 17 image, which is a runtime environment for running Java applications
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME # set up the working directory for the build (where all subsequent commands will be run)
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar # copy the built JAR file from the build stage to the run stage, and rename it myapp.jar.

ENTRYPOINT java -jar myapp.jar # the container should run the Java application by executing java -jar myapp.jar when the container starts
```

We need a multistage build to separate the build environment from the runtime environment. With this method we get a smaller, more secure and more efficient Docker image because we only have the necessary artifacts in our final image, without all the useless features.

### 1-3. Document docker-compose most important commands. 

Docker Compose Commands Explained
1. **docker compose up**

    This command creates and starts the containers defined in the docker-compose.yml file. If the images are not already built, it builds them first.

    ```sh
    docker compose up
    docker compose up -d  # Run in detached mode
    docker compose up --force-recreate # Force to recreate the containers, even if they already exist
    ```

2. **docker compose down**

    Cleans up all resources created by docker-compose up.

    ```sh
    docker compose down
    ```

3. **docker compose build**

    Builds the images defined in the docker-compose.yml file.

    ```sh
    docker compose build
    ```

4. **docker compose stop**

    Stops the services defined in the docker-compose.yml file but keeps the containers, networks, and volumes intact.

    ```sh
    docker compose stop
    ```


5. **docker compose start**

    Starts the services that were stopped using docker-compose stop.

    ```sh
    docker compose start
    ```

6. **docker compose restart**

    Stops and then starts services, useful for applying changes without recreating containers.

    ```sh
    docker compose restart
    ```

7. **docker compose ps**

    Displays the status of services defined in the docker-compose.yml file.

    ```sh
    docker compose ps
    ```

8. **docker compose logs**

    Shows the logs for the services defined in the docker-compose.yml file.

    ```sh
    docker compose logs
    ```

9. **docker compose exec**

    Runs a specified command in a service's container, similar to docker exec.

    ```sh
    docker compose exec <service_name> <command>
    ```

10. **docker compose rm**

    Removes stopped containers created by docker-compose up, not affecting any running containers.

    ```sh
    docker compose rm
    ```

11. **docker compose config**

    Helps to verify the syntax and structure of the docker-compose.yml file.

    ```sh
    docker compose config
    ```

12. **docker compose pull**

    Pulls the latest images for services defined in the docker-compose.yml file from the registry.

    ```sh
    docker compose pull
    ```

13. **docker compose stats**

     Provides a live view of resource usage (CPU, memory, network, etc.) for each service defined in the docker-compose.yml file.

    ```sh
    docker compose stats
    ```

14. **docker compose inspect**

    Shows detailed configuration and status information about the specified service or container.

    ```sh
    docker compose inspect <service_name>
    ```

### 1-4. Document your docker-compose file.

Here is my docker-compose.yml file : 

```yml
version: '3.7' # Version of docker-compose

services:
    backend:
        container_name: backend # Set the name of the container
        build:
            context: ./API/simple-api-student
            dockerfile: Dockerfile
            # Build the image using the Dockerfile located in ./API/simple-api-student
        image: theoburdinat/myapi # Name the image as "theoburdinat/myapi"
        environment: # Set environment variables to make the connection between the database and the API service
            DB_host: database
            DB_port: 5432
            DB_name: db 
            DB_user: theo
            DB_pass: pa$$
        networks: # Connect to the same network as the other containers
            - app-network
        depends_on: # Ensure that the database service is started before the backend service
            - database

    database:
        container_name: database
        build:
            context: ./postgres
            dockerfile: Dockerfile
        image: theoburdinat/mydatabase
        environment: # Environment variables to set up the postgreSQL database
            POSTGRES_USER: theo
            POSTGRES_PASSWORD: pa$$
        networks:
            - app-network
        volumes: # Use a volume to keep our data in all circumstances
            - ./postgres/data:/var/lib/postgresql/data

    httpd:
        container_name: httpd
        build:
            context: ./HTTP
            dockerfile: Dockerfile
        image: theoburdinat/myhttpd
        environment: # Environment variable to make a good connection between the httpd service and the API service
            BACKEND_HOST: backend
        ports: # Map port 80 on the host to port 80 in the container 
            - "80:80"
        networks:
            - app-network
        depends_on:
            - backend

networks:
    app-network:
    # Define a custom network named app-network for inter-service communication

volumes:
    data:
    # Define a volume named "data" for persistent storage
```


### 1-5. Document your publication commands and published images in dockerhub.

We can publish our images to make them available for colleagues or other machines. To do so we have to do the following commands :

- `docker login` - to login to your DockerHub account.
- `docker tag <image name> <image name>:<X.X version>` - to version our images
- `docker push <image name>:<X.X version>` - to push your image on DockerHub.

Then you can see your images on [DockerHub](https://hub.docker.com/). I uploaded `theoburdinat/myapi`, `theoburdinat/myhttpd` and `theoburdinat/mydatabase` on DockerHub.

## PW n°2

### 2-1 What are testcontainers?

.

