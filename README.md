# Overview - Udagram Image Filtering Microservice
The project application, **Udagram** - an Image Filtering application, allows users to register and log into a web client, post photos to the feed, and process photos using an image filtering microservice. It has two components:
1. Frontend - Angular web application built with Ionic framework
2. Backend RESTful API - Node-Express application

We will learn more when we will convert the application into microservices.

# Prerequisites
You should have the following tools installed in your local machine:

* <a href="https://git-scm.com/downloads" target="_blank">Git</a> for Mac/Linux/Windows. 
>Windows users: Once you download and install Git for Windows, you can execute all the bash, ssh, git commands in the **Gitbash** terminal. Whereas Windows users using [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/about) (WSL) can follow all steps as if they are Linux users.


* The following will help you run your project locally as a monolithic application.
   1. PostgreSQL **client**, the `psql` command line utility, installed locally. 
We will set the PostgreSQL server up in the AWS cloud. The client will help you to connect with the server. Usually, the client comes along with the [PostgreSQL](https://www.postgresql.org/download/) server installation, but you can install only the client using:
```bash
# Mac OS
brew install libpq  
brew link --force libpq 
# Ubuntu
sudo apt-get install postgresql-client
# Windows, you need to install the complete server
```
Otherwise, see the complete (server and client) installation instructions for [Mac](https://www.postgresqltutorial.com/install-postgresql-macos/), [Linux](https://www.postgresqltutorial.com/install-postgresql-linux/), and [Windows](https://www.postgresqltutorial.com/install-postgresql/). 
   2. <a href="https://nodejs.org/en/download/" target="_blank">NodeJS</a> v12.14 or higher - NodeJS installer will install both Node.js and npm on your system. Verify using the commands:
```bash
# v12.14 or higher
node -v 
# v7.19 or higher
npm -v
# You can upgrade to the latest version of npm using:
npm install -g npm@latest
```
   3. [Ionic command-line utility v6](https://ionicframework.com/docs/installation/cli) or higher framework to build and run the frontend application locally. Verify the installation as:
```bash
# v6.0 or higher
ionic --version
# Otherwise, install a fresh version using
npm install -g @ionic/cli
```

* <a href="https://docs.docker.com/desktop/#download-and-install" target="_blank">Docker Desktop</a> for running the project locally in a multi-container environment

* <a href="https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html" target="_blank">AWS CLI v2</a> for interacting with AWS services via your terminal. After installing the AWS CLI, you will also have to configure the access profile locally. 
   * Create an IAM user with Admin privileges on the AWS web console. Copy its Access key. 
   * Configure the access profile locally using the Access key generated above:
   ```bash
   aws configure [--profile nd9990]
   ```
* <a href="https://kubernetes.io/docs/tasks/tools/#kubectl" target="_blank">Kubectl</a> command-line utility to create and communicate with Kubernetes clusters

In addition to the tools above, fork and then clone the project starter code from the <a href="https://github.com/udacity/nd9990-c3-microservices-exercises/tree/master/project" target="_blank">Udacity GitHub repository</a>.

<br data-md>

# Getting started

Set up the resources that you will need while running the application either locally or on the cloud. 

### Set up an S3 bucket to store the user uploaded data

1. Create a public S3 bucket with default configuration, such as no versioning and disabled encryption. 
1. Add bucket policy allowing other AWS services (Kubernetes) to access the bucket contents. You can use the <a href="https://awspolicygen.s3.amazonaws.com/policygen.html" target="_blank">policy generator</a> tool to generate such an IAM policy. See an example below (change the bucket name in your case).
```json
{
"Version": "2012-10-17",
"Statement": [
{
"Sid": "Stmt1625306057759",
"Principal": "*",
"Action": "s3:*",
"Effect": "Allow",
"Resource": "arn:aws:s3:::test-nd9990-dev-wc"
}
]
}
```
1. Add the <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/ManageCorsUsing.html#cors-example-1" target="_blank">CORS configuration</a> to allow the application running outside of AWS to interact with your bucket. You can use the following configuration:
```json
[
    {
        "AllowedHeaders": [
            "*"
        ],
        "AllowedMethods": [
            "POST",
            "GET",
            "PUT",
            "DELETE",
            "HEAD"
        ],
        "AllowedOrigins": [
            "*"
        ],
        "ExposeHeaders": []
    }
]
```
Note: In the S3 console, the CORS configuration must be JSON format. Whereas, the CLI can use either JSON or XML format.
1. Once the policies above are set, you can disable public access to your bucket.


### Set up AWS RDS - PostgreSQL database 
You will access this database from your application running either locally or on the cloud. 

1. Navigate to the <a href="https://console.aws.amazon.com/rds/home" target="_blank">RDS dashboard</a> and create a PostgreSQL database with the following configuration, and leave the remaining fields as default.

<center>

|**Field**|**Value**|
|---|---|
|Database creation method|**Standard create**. <br>Easy create option creates <br>a private database by default. |
|Engine option|PostgreSQL 12 or higher|
| Templates |Free tier|
| DB instance identifier, <br>master username, and password|Your choice|
|DB instance class|Burstable classes with minimal size |
|VPC and subnet |Default|
|Public access|YES. Allow application running outside <br> of your AWS account discover the database.|
|VPC security group|Either choose default or <br>create a new one|
| Availability Zone|No preferencce|
|Database port|`5432` (default)|
</center>
2. Once the database is created successfully, copy and save the database endpoint, master username, and password to your local machine. It will help your application discover the database. 

3. Edit the security group's inbound rule to allow incoming connections from anywhere (`0.0.0.0/0`). It will allow your application running locally connecting to the database. 

4. Test the connection from your local PostgreSQL client.
```bash
# Assuming the endpoint is: mypostgres-database-1.c5szli4s4qq9.us-east-1.rds.amazonaws.com
psql -h mypostgres-database-1.c5szli4s4qq9.us-east-1.rds.amazonaws.com -U [your-username] postgres
# Provide the database password that you had set in the step above
# It will open the "postgres=>" prompt if the connection is successful
```
Later, when your application up and running, you can run commands like:
```bash
# List the databases
\list
# Go inside the "postgres" database and view relations
\c postgres
\dt
   ```


### Set up the Environment variables
1. If not already, fork the project repository and clone it.
```bash
git clone https://github.com/[Github-Username]/nd9990-c3-microservices-exercises.git
cd nd9990-c3-microservices-exercises/project/
```

2. Your application will need to access the AWS PostgreSQL database and S3 bucket you created in the steps above. The connection details (confidential) of the database and S3 bucket should not be hard-coded into the application code. 

 For this reason, create and stores the above details into multiple environment variables locally. 
   - **Mac/Linux users** - Use the *set_env.sh* file present in the project directory to configure these variables on your local machine. Once you save your details in the *set_env.sh* file, run:
```bash
# Mac/Linux - Load the environment variables
source set_env.sh
``` 
Also, you would not want your credentials to be stored in the Git repository either. Run the following command at the end, to tell git to stop tracking the script in git but keep it stored locally.
```bash
git rm --cached set_env.sh
```
 In addition, add the *set_env.sh*  filename to your `.gitignore` file in the project repository. <br><br>

   - **Mac/Linux users** - Setting the environment variables the way, as shown above, is not permanent. **Every time you open a new terminal, you will have to run `source set_env.sh` to reconfigure your environment variables** and verify with a command like `echo $POSTGRES_USERNAME`. The solution is to save all the variables above in your `~/.profile` file and use:
```bash
source ~/.profile
```

   - **Windows users** - Set all the environment variables as shown in the *set_env.sh* file either using the **Advanced System Settings** or run the following in the GitBash terminal (change the values, as applicable to you):
```bash
setx POSTGRES_USERNAME postgres
setx POSTGRES_PASSWORD abcd1234
setx POSTGRES_HOST mypostgres-database-1.c5szli4s4qq9.us-east-1.rds.amazonaws.com
setx POSTGRES_DB postgres
setx AWS_BUCKET test-nd9990-dev-wc
setx AWS_REGION us-east-1
setx AWS_PROFILE nd9990
setx JWT_SECRET hello
setx URL http://localhost:8100
```


# Part 1 - Run the project locally as a Monolithic application

Now that you have set up the AWS PostgreSQL database and S3 bucket, and saved the environment variables, let's run the application locally.  It's recommended that you start the backend application first before starting the frontend application that depends on the backend API.

### Backend App

1. Download all the package dependencies by running the following command from the */project/udagram-api/* directory:
```bash
npm install .
```
1. Run the application locally, in a development environment (so that you can test your edits quickly without restarting the server):
```bash
npm run dev
```
If the command above runs successfully, visit the *http://localhost:8080/api/v0/feed* in your web browser to verify that the application is running. You should see a JSON payload. 


### Frontend App

3. To download all the package dependencies, run the command from the */project/udagram-frontend/* directory: 
```bash
npm install .
```
3. Prepare your application by compiling them into static files. 
```bash
# You should have the Ionic command-line utility installed before you run the command below. 
# Otherwise visit https://ionicframework.com/docs/intro/cli
ionic build
```
3. Run the application locally using files created from the `ionic build` command above.
```bash
ionic serve
```
3. Visit *http://localhost:8100* in your web browser to verify that the application is running. You should see a web interface.



### Optional

7. It's useful to "lint" your code so that changes in the codebase adhere to a coding standard. This helps alleviate issues when developers use different styles of coding. `eslint` has been set up for TypeScript in the codebase for you. To lint your code, run the following:
```bash
npx eslint --ext .js,.ts src/
```
To have your code fixed automatically, run
```bash
npx eslint --ext .js,.ts src/ --fix
```
7. Over time, our code will become outdated and inevitably run into security vulnerabilities. To address them, you can run:
```bash
npm audit fix
```


# Part 2 - Run the project locally in a multi-container environment

The objective of this part of the project is to:

* Refactor the monolith application to microservices
* Set up each microservice to be run in its own Docker container

Once you refactor the Udagram application, it will have the following services running internally:

1. Backend `/user/` service - allows users to register and log into a web client.
1. Backend `/feed/` service - allows users to post photos, and process photos using image filtering. 
1. Frontend - It is a basic Ionic client web application that acts as an interface between the user and the backend services.
1. Nginx as a reverse proxy server - for resolving multiple services running on the same port in separate containers. When different backend services are running on the same port, then a reverse proxy server directs client requests to the appropriate backend server and retrieves resources on behalf of the client.

> Keep in mind that we don’t want to make any feature changes to the frontend or backend code. If a user visits the frontend web application, it should look the same regardless of whether the application is structured as a monolith or microservice.  
>


Navigate to the project directory, and set up the environment variables again:

```bash
source set_env.sh
```

### Refactor the Backend API

The current */project/udagram-api/* backend application code contains logic for both */users/*  and */feed/*  endpoints. Let's decompose the API code into the following two separate services that can be run independently of one another.

1. */project/udagram-api-feed/* 
1. */project/udagram-api-user/* 

Create two new directories (as services) with the names above. Copy the backend starter code into the above individual services, and then break apart the monolith. Each of the services above will have the following directory structure, with a lot of duplicate code.

```bash
.
├── mock           # Common and no change 
├── node_modules   # Auto generated. Do not copy. Add this into the .gitignore and .dockerignore
├── package-lock.json # Auto generated. Do not copy.
├── package.json      # Common and no change 
├── src
│   ├── config        # Common and no change
│   ├── controllers/v0  # TODO: Keep either /feed or /users service.   Delete the other folder
│         ├── index.router.ts  # TODO: Remove code related to other (either feed or users) service 
│         └── index.router.ts  # TODO: Remove code related to other (either feed or users) service 
│   ├── migrations   # Common and no change
│   ├── aws.ts       # Common and no change
│   ├── sequelize.ts # Common and no change
│   └── server.ts    # TODO: Remove code related to other (either feed or users) service 
├── Dockerfile       # TODO: Create NEW, and common
├── .gitignore       # Common and no change
├── .dockerignore    # TODO: Add "node_modules" to this file
└── migrations       # TODO: Remove the JSON related to other (either feed or users) service  
```

The Dockerfile for the above two backend services will be like:

```bash
FROM node:13
# Create app directory
WORKDIR /usr/src/app
# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./
RUN npm ci 
# Bundle app source
COPY . .
EXPOSE 8080
CMD [ "npm", "run", "prod" ]
```
It's not a hard requirement to use the exact same Dockerfile above. Feel free to use other base images or optimize the commands. 

<br data-md>

### Refactor the Frontend Application

In the frontend service, you just need to add a different Dockerfile and `.dockerignore` to the */project/udagram-frontend/* directory. 

```bash
## Build
FROM beevelop/ionic:latest AS ionic
# Create app directory
WORKDIR /usr/src/app
# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
COPY package*.json ./
RUN npm ci
# Bundle app source
COPY . .
RUN ionic build
## Run 
FROM nginx:alpine
#COPY www /usr/share/nginx/html
COPY --from=ionic  /usr/src/app/www /usr/share/nginx/html
```


### How would containers discover each other and communicate?
Use another container named *reverseproxy* running the Nginx server. The *reverseproxy* service will help add another layer between the frontend and backend APIs so that the frontend only uses a single endpoint and doesn't realize it's deployed separately. *This is one of the approaches and not necessarily the only way to deploy the services. *To set up the *reverseproxy* container, follow the steps below:

1. Create a newer directory */project/udagram-reverseproxy/  *
2. Create a Dockerfile as:
```bash
FROM nginx:alpine
COPY nginx.conf /etc/nginx/nginx.conf
```

3. Create the *nginx.conf* file that will associate all the service endpoints as:
```bash
worker_processes 1;  
events { worker_connections 1024; }
error_log /dev/stdout debug;
http {
    sendfile on;
    upstream user {
        server backend-user:8080;
    }
    upstream feed {
        server backend-feed:8080;
    }
    proxy_set_header   Host $host;
    proxy_set_header   X-Real-IP $remote_addr;
    proxy_set_header   X-NginX-Proxy true;
    proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header   X-Forwarded-Host $server_name;    
    server {
        listen 8080;
        location /api/v0/feed {
            proxy_pass         http://feed;
        }
        location /api/v0/users {
            proxy_pass         http://user;
        }            
    }
}
```
The Nginx container will expose 8080 port. The configuration file above, in the `server` section, it will route the *http://localhost:8080/api/v0/feed* requests to the *backend-user:8080* container. The same applies for the *http://localhost:8080/api/v0/users* requests.




### Use Docker compose to build and run multiple Docker containers

> **Note**: The ultimate objective of this step is to have the Docker images for each microservice ready locally. This step can also be done manually by building and running containers one by one. 


1. Once you have created the Dockerfile in each of the following services directories, you can use the `docker-compose` command to build and run multiple Docker containers at once.

   - */project/udagram-api-feed/* 
   - */project/udagram-api-feed/* 
   - */project/udagram-frontend/* 
   - */project/udagram-reverseproxy/*

 The `docker-compose`  <a href="https://docs.docker.com/compose/" target="_blank">command</a> uses a YAML file to configure your application’s services in one go. Meaning, you create and start all the services from your configuration file, with a single command. Otherwise, you will have to individually build containers one-by-one for each of your services. 



2. **Create images** - In the project's parent directory, create a [docker-compose-build.yaml](https://video.udacity-data.com/topher/2021/July/60e28b72_docker-compose-build/docker-compose-build.yaml) file with the following content. It will create an image for each individual service. Then, you can run the following command to create images locally:
```bash
# Make sure the Docker services are running in your local machine
# Remove unused and dangling images
docker image prune --all
# Run this command from the directory where you have the "docker-compose-build.yaml" file present
docker-compose -f docker-compose-build.yaml build --parallel
```
>**Note**: YAML files are extremely indentation sensitive, that's why we have attached the files for you. 


3. **Run containers** using the images created in the step above. Create another YAML file, [docker-compose.yaml](https://video.udacity-data.com/topher/2021/July/60e28b91_docker-compose/docker-compose.yaml),  in the project's parent directory. It will use the existing images and create containers. While creating containers, it defines the port mapping, and the container dependency. 

 Once you have the YAML file above ready in your project directory, you can start the application using:
```bash
docker-compose up
```

4. Visit http://localhost:8100 in your web browser to verify that the application is running. 


### Troubleshoot

1. Make sure that the environment variables are set correctly in your terminal. Try using the `echo $URL` command to see the intended value. 


2. CORS related errors, such as:
   - Http failure response for http://localhost:8080/api/v0/users/auth/: 0 Unknown Error
   - Access to XMLHttpRequest at 'http://localhost:8080/api/v0/users/auth/login' from origin 'http://localhost:8100' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
   - Access to XMLHttpRequest at 'http://localhost:8080/api/v0/feed' from origin 'http://localhost:8100' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource.

 Ensure that the environment variables are set correctly while running the containers. Check using:
```bash
# Run from the directory where you have the compose file present
docker-compse config
```
In addition, verify in your AWS console that
 - S3 has the the CORS policy enabled
 - Check the RDS logs


3. If you are rebuilding the images, you must delete the existing images locally, using:
```bash
# Run from the directory where you have the compose file present
docker-compose down
# delete everything
docker system prune -a --volumes
# To delete all dangling images
docker image prune --all
```

4. When migrating this application to use containers, we may run into an issue with the `bcrypt` package. This is because the *node_modules* were installed on an operating system different than that of the one in the Docker image. The solution is to set `node_modules` in our `.dockerignore` file so that *node_modules* are not copied over from our local machine into the container.


5. Lastly, if you local containers are still not able to communicate through, it's possibly because the backend, and the Nginx containers are not on the same network. As long as your  http://localhost:8100 (fetching results from the S3 bucket) and http://localhost:8080/api/v0/feed (returning the JSON output) are running fine in the browser, you should be all right. 



# Part 3 - Set up Travis continuous integration pipeline

Before you move on to the next step, deploying the application to the Kubernetes cluster, you need to set up a CI pipeline to build and push the images to the Dockerhub automatically (on commit to Github). 

Having images present in the Dockerhub will help to deploy the multi-container application to your Kubernetes cluster.

<br data-md>

### Create Dockerhub repositories

Log in to https://hub.docker.com/ and create four public repositories - each repository corresponding to your local Docker images. The names of the repositories must be exactly the same as the **image name** specified in the *docker-compose-build.yaml* file:

* reverseproxy
* udagram-api-user
* udagram-api-feed
* udagram-frontend


### Set up Travis CI Pipeline

Use Travis CI pipeline to build and push images to your DockerHub registry. 

1. Sign in to https://travis-ci.com/ (not https://travis-ci.org/) using your Github account (preferred).


2. Integrate Github with Travis: Activate your GitHub repository with whom you want to set up the CI pipeline. 


3. Set up your Dockerhub username and password in the Travis repository's settings, so that they can be used inside of `.travis.yml` file while pushing images to the Dockerhub. 


4. Add a `.travis.yml` configuration file to the project directory (locally). It should automatically read the Dockerfiles, build images, and push images to DockerHub. Your Travis file must have the logic for the following, in addition to the mandatory sections: 
```bash
# Assuming the .travis.yml file is in the project directory, and there is a separate sub-directory for each service
# Build
  - docker build -t udagram-api-feed ./udagram-api-feed
  - docker build -t udagram-api-user ./udagram-api-user
  - docker build -t reverseproxy ./udagram-reverseproxy
  - docker build -t udagram-frontend ./udagram-frontend
```
```bash
# Tagging
  - docker tag udagram-api-feed sudkul/udagram-api-feed:v1
  - docker tag udagram-api-user sudkul/udagram-api-user:v1
  - docker tag reverseproxy sudkul/reverseproxy:v1
  - docker tag udagram-frontend sudkul/udagram-frontend:v1
```
```bash
# Push
  - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
  - docker push sudkul/udagram-api-feed:v1
  - docker push sudkul/udagram-api-user:v1
  - docker push sudkul/reverseproxy:v1
  - docker push sudkul/udagram-frontend:v1
```
> **Tip**: Use different tags each time you push images to the Dockerhub.   


5. Trigger your build by pushing your changes to the Github repository. All of these steps mentioned in the `.travis.yml` file will be executed on the Travis worker node. It may take upto 15-20 minutes to build and push all four images.


6. Verify if the newly pushed images are now available in your Dockerhub account.


### Troubleshoot

If you are not able to get through the Travis pipeline, and still want to push your local images to the Dockerhub (only for testing purposes), you can attempt the manual method. However, this is only for the troubleshooting purposes, such as **verifying the deployment to the Kubernetes cluster. **

* Log in to the Docker from your CLI, and tag the images with the name of your registry name (Dockerhub account username). 
```bash
# See the list of current images
docker images
# Use the following syntax
# In the remote registry (Dockerhub), we can have multiple versions of an image using "tags". 
# docker tag <local-image-name:current-tag> <registry-name>/<repository-name>:<new-tag>
docker tag <local-image:tag> <dockerhub-username>/<repository>:<tag>
```
* Push the images to the Dockerhub. 
```bash
docker login --username=<your-username>
# Remove unused and dangling images
docker image prune --all
# Use the "docker push" command for each image, or 
# Run this command from the directory where you have the "docker-compose-build.yaml" file present
docker-compose -f docker-compose-build.yaml build --parallel
# Use "docker-compose -f docker-compose-build.yaml push" if the names in the compose file are as same as the Dockerhub repositories. 
docker-compose -f docker-compose-build.yaml push
```


# Part 4 - Container Orchestration with Kubernetes

For the current part, verify that you have the `kubectl` utility installed locally:
```bash
kubectl version --short
# Returns: Client Version: v1.21.2
```
Optionally, you can use the <a href="https://eksctl.io/introduction/#installation" target="_blank">EKSCTL</a> utility to create an EKS cluster. Otherwise, you can create one via the AWS web console. 


### Create a Kubernetes cluster in AWS EKS service

1. **Deployment configuration:** Create a single *deployment.yaml* file either for the entire project or you can create a separate *deployment.yaml* file for each service. While defining container specs, make sure to specify the same images you've pushed to the Dockerhub earlier. Ultimately, the frontend web application and backend API applications should run in their respective pods.


2. **Service configuration: **Similarly, create the *service.yaml* file thereby defining the right services/ports mapping.


3. Create a** public **Kubernetes cluster and create and attach the nodegroup to the cluster. Feel free to choose the nodegroup size and configuration as you find suitable. You can do this using either the AWS web console or AWS CLI. 


4. [Optional] Use EKSCTL, a command-line utility, to create an EKS cluster and the associated AWS resources (IAM roles) in a single command. 
```bash
# Feel free to use the same/different flags as you like
eksctl create cluster --name myCluster --region=us-east-1 --version=1.18 --nodes-min=2 --nodes-max=3
# Recommended: You can see many more flags using "eksctl create cluster --help" command.
# For example, you can set the node instance type using --node-type flag
```
The default command above will set the following for you:
  - An auto-generated name
  - Two m5.large worker nodes. Recall that the worker nodes are the virtual machines, and the m5.large type defines that each VM will have 2 vCPUs, 8 GiB memory, and up to 10 Gbps network bandwidth.
  - Use the Linux AMIs as the underlying machine image
  - An autoscaling group with [2-3] nodes
  - Your default region
  - A default VPC
  - Importantly, it will write cluster credentials to the default config file locally. Meaning, EKSCTL will set up KUBECTL to communicate with your cluster. If you'd have created the cluster using the web console, you'll have to set up the *kubeconfig* manually. 

 ```bash
 # Once you get the success confirmation, run
 kubectl get nodes
 ```
> Known issue: Sometimes, the cluster creation may fail in the **us-east-1** region. In such a case, use `--region=us-east-2` flag.

 If you run into issues, either go to your CLoudFormation console, or run:
```bash
eksctl utils describe-stacks --region=us-east-1 --cluster=myCluster
```


### Deployment

In this step, you will deploy the Docker containers for the frontend web application and backend API applications in their respective pods.

Recall that while splitting the monolithic app into microservices, you used the values saved in the environment variables, as well as AWS CLI was configured locally. Similar values are required while instantiating containers from the Dockerhub images. 

1. **ConfigMap:** Create *env-configmap.yaml*, and save all your configuration values (non-confidential environments variables) in that file. 


2. **Secret: **Do not store the PostgreSQL username and passwords in the *env-configmap.yaml* file. Instead, create *env-secret.yaml* file to store the confidential values, such as login credentials. 


3. **Secret: **Create *aws-secret.yaml* file to store your AWS login credentials. Replace `___INSERT_AWS_CREDENTIALS_FILE__BASE64____` with the Base64 encoded credentials (not the regular username/password). 
     * Mac/Linux users: If you've configured your AWS CLI locally, check the contents of *~/.aws/credentials* file using `cat ~/.aws/credentials` . It will display the *aws_access_key_id* and *aws_secret_access_key* for your AWS profile(s). Now, you need to select the applicable pair of *aws_access_key* from the output of the `cat` command above and convert that string into `base64` . You use commands, such as:
```bash
# Use a combination of head/tail command to identify lines you want to convert to base64
# You just need two correct lines: a right pair of aws_access_key_id and aws_secret_access_key
cat ~/.aws/credentials | tail -n 5 | head -n 2
# Convert 
cat ~/.aws/credentials | tail -n 5 | head -n 2 | base64
```
     * **Windows users:** Copy a pair of *aws_access_key* from the AWS credential file and paste it into the encoding field of this third-party website: https://www.base64encode.org/ (or any other). Encode and copy/paste the result back into the *aws-secret.yaml*  file.

<br data-md>


4. **Deployment configuration:** Create *deployment.yaml* file individually for each service. While defining container specs, make sure to specify the same images you've pushed to the Dockerhub earlier. Ultimately, the frontend web application and backend API applications should run in their respective pods.

5. **Service configuration: **Similarly, create the *service.yaml* file thereby defining the right services/ports mapping.


Once, all deployment and service files are ready, you can use commands like:
```bash
# Apply env variables and secrets
kubectl apply -f aws-secret.yaml
kubectl apply -f env-secret.yaml
kubectl apply -f env-configmap.yaml
# Deployments
kubectl apply -f backend-feed-deployment.yaml
kubectl apply -f backend-user-deployment.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f reverseproxy-deployment.yaml
# Service
kubectl apply -f backend-feed-service.yaml
kubectl apply -f backend-user-service.yaml
kubectl apply -f frontend-service.yaml
kubectl apply -f reverseproxy-service.yaml
```
Make sure to check the image names in the deployment files above. 


### Expose External IP

Use this link to <a href="https://kubernetes.io/docs/tutorials/stateless-application/expose-external-ip-address/" target="_blank">expose an External IP</a> address to access your application in the EKS Cluster.

```bash
kubectl get deployments
# Create a Service object that exposes the deployment:
kubectl expose deployment frontend --type=LoadBalancer --name=publicfrontend
kubectl get services publicfrontend
# Note down the External IP, such as 
# a5e34958a2ca14b91b020d8aeba87fbb-1366498583.us-east-1.elb.amazonaws.com
# Similarly, do it for the reverseproxy deployment
kubectl expose deployment reverseproxy --type=LoadBalancer --name=public reverseproxy
kubectl get services publicreverseproxy
# Note down the External IP, such as
# a846b8379f2c147f0b4c9ba9da9bfc09-795781586.us-east-1.elb.amazonaws.com
```

<br data-md>

### Update the environment variables 

Once you have the External IP of your front end and reverseproxy deployment, Change the API endpoints in the following places locally:

* Environment variables - Replace the http://**localhost**:8100 string with the External IP of the frontend.  After replacing run `source ~/.zshrc` and verify using `echo $URL`

*  *udagram-deployment/env-configmap.yaml* file - Replace http://localhost:8100 string with the External IP of the frontend. 

* *udagram-frontend/src/environments/environment.ts* file - Replace 'http://localhost:8080/api/v0' string with the External IP of the reverseproxy. Alternatively you can use the ClusterIP of the *reverseproxy* deployment.  

*  *udagram-frontend/src/environments/environment.prod.ts* - Replace 'http://localhost:8080/api/v0' string. 

* ** [Optional]** Re tag in the `.travis.yaml` (say use `:v3`) as well as deployment YAML files

Then, push your changes to the Github repo. Travis will automatically build and re-pushed images to your Dockerhub. Next, re-apply configmap and redeploy to k8s cluster.

```bash
kubectl apply -f env-configmap.yaml
kubectl set image deployment frontend frontend=sudkul/udagram-frontend:v2
kubectl set image deployment backend-feed backend-feed=sudkul/udagram-api-feed:v1
kubectl set image deployment backend-user backend-user=sudkul/udagram-api-user:v1
kubectl set image deployment reverseproxy reverseproxy=sudkul/reverseproxy:v2
```
Check your deployed application at the External IP of your frontend deployment. 




### Troubleshoot

1. If you see *Failed to load resource: net::ERR_CONNECTION_REFUSED* in your browser after deployment to the k8s cluster,  it means you may not have set the environment variables correctly.
2. Use this command to see the STATUS of your pods:
```bash
kubectl get pods
```
In case of `ImagePullBackOff` or `ErrImagePull`, review your deployment.yaml file(s) if they have the right image path. 


### Verify/Screenshots

So that we can verify that your project is deployed, please include the screenshots of the following commands with your completed project. 
```bash
# Kubernetes pods are deployed properly
kubectl get pods 
# Kubernetes services are set up properly
kubectl describe services
# You have horizontal scaling set against CPU usage
kubectl describe hpa
```


### Clean up
1. Delete the EKS cluster. If you have used the EKSCTL utility, then use:
```bash
eksctl delete cluster --name=myCluster
```
2. Delete the S3 bucket and RDS PostgreSQL database. 



