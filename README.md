## Jenkins Python Build Test Demo
### Installation
Installing Jenkins Using Docker Tutorial: https://www.jenkins.io/doc/book/installing/docker/
- Create a bridge network in Docker:
	- `docker network create jenkins`
- To verify it: 
	- `docker network ls`
- Create the Dockerfile
```
		FROM jenkins/jenkins:2.440.1-jdk17
		USER root
		RUN apt-get update && apt-get install -y lsb-release
		RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
		  https://download.docker.com/linux/debian/gpg
		RUN echo "deb [arch=$(dpkg --print-architecture) \
		  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
		  https://download.docker.com/linux/debian \
		  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
		RUN apt-get update && apt-get install -y docker-ce-cli
		USER jenkins
		RUN jenkins-plugin-cli --plugins "blueocean docker-workflow"
```

- Build a new docker image from this Dockerfile and assign the image a meaningful name, e.g. "myjenkins-blueocean:2.440.1-1":
	- `docker build -t myjenkins-blueocean:2.440.1-1 .` 
- If you have not yet downloaded the official Jenkins Docker image, the above process automatically downloads it for you.

- Run your own myjenkins-blueocean:2.440.1-1 image as a container in Docker using the following docker run command:
	
```
	docker run --name jenkins-blueocean --restart=on-failure --detach \
	 --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
	 --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
	 --volume jenkins-data:/var/jenkins_home \
	 --volume jenkins-docker-certs:/certs/client:ro \
	 --publish 8080:8080 --publish 50000:50000 myjenkins-blueocean:2.440.1-1
```

- Check if docker container is running: 
	- `docker ps`

- Browse to http://localhost:8080 (or whichever port you configured for Jenkins when installing it) and wait until the **Unlock Jenkins** page appears. 
- To unlock Jenkins, you need to retrieve the administrator password from the specified file. In our case, the password is stored in the file `/var/jenkins_home/secrets/initialAdminPassword` within the Jenkins home directory.
- Access the Jenkins container: `docker exec -it jenkins-blueocean sh`
- Once inside the container, navigate to the directory containing the password file: `cd /var/jenkins_home/secrets`
- View the content of the initialAdminPassword file: `cat initialAdminPassword`
	- Password: b79f30de0329417796e357ed160d1066
- Install Jenkins with the suggested plugins.
- Create First Admin User
- Instance Configuration: leave the default Jenkins URL.

- Troubleshoot: If in the notification bell appears the following message: "Some plugins could not be loaded due to unsatisfied dependencies. Fix these issues and restart Jenkins to re-enable these plugins." then:
	- Manage Jenkins
	- System Configuration: Plugins
	- Updates
		- Select all and update
	- Download progress: click on "Restart Jenkins when installation is complete and no jobs are running".

### Setting up Declarative Pipelines using Groovy
Useful tutorials: 
- [Jenkins Python Pipeline Tutorial](https://www.youtube.com/watch?v=6njM8g5hKuk)
- [Learn Jenkins! Complete Jenkins Course - Zero to Hero](https://www.youtube.com/watch?v=6YZvp2GwT0A)

Steps:
- New Item
- Pipeline
- Enter an item name
#### Option 1: Create the pipeline script directly into the Jenkins UI
- Pipeline
	- Pipeline script
		- write your pipeline here
	- Pipeline Syntax:
		- Checkout
			- checkout: Check out from version control
			- SCM: Git
			- Repository URL
			- Generate pipeline script
		- Build
			- Sample Step: sh:Shell Script
			- Inside the build stage we need to again get the command to grab the branch for the code that we want to be testing. 
		- Test
			- Sample Step: sh:Shell Script

- To see if Python is installed inside the Docker container: 
	- Access the Docker container as a root user where Jenkins is running: docker exec -it -u 0 jenkins-blueocean bash
	- python3
	- If not installed: 
		- Update the package repository: apt-get update
		- Install Python: apt-get install -y python3
		- After the installation is complete, verify that Python is installed by running the following command: python3 --version
	- To exit from inside a Docker container's shell: exit
- The 'source' directory is structured as a Python package with an __init__.py file inside it. This allows Python to recognize it as a module. If it's not already a package, create an empty __init__.py file inside the 'source' directory.
- If pip is not installed inside the Docker container:
	-  apt-get install python3-pip 
- If pytest is not installed: 
	- apt install python3-pytest
#### Option 2: Create the pipeline script using a Jenkins file (preferred method)
- We have a Jenkins file in the source repository (GitHub repository) and we point this pipeline over to that Jenkins file and it takes in our pipeline script there.
- Configuration --> Pipeline
- Pipeline script from SCM
- SCM: Git
- Repository URL: https://github.com/bernatsort/jenkins_python_build_test_demo.git
- Branch Specifier (blank for 'any'): main
- Script Path: directory where we have the Jenkinsfile (in this case Jenkinsfile).
- Now, for future builds, next time we do a git commit and push to GitHub, this build should trigger automatically. 
- Go to Console Output (Click the tick icon in the Build history).
- Then, Open Blue Ocean. The pipeline has all the different stages. If we click on each individual step we can see exactly what happened in these stages. Blue Ocean is pretty good for giving us a nice interface so we can go and look at the build and look what happened in each of stages of the build. 