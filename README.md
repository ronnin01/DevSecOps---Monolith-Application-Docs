

# Basic DevSecOps Monolith Laravel API Application

This is a step by step setup of DevSecOps of Monolith application. If you're starting to learn DevOps this is a good basic practices on developing application with CI/CD integration and Code Analysis and Monitoring application. We Will setup a simple application using **Laravel** with **Docker** for development, **Jenkins** with **SonarQube** and **Trivy** for CI/CD automation and Code Analysis and also you can use **Kubernetes** along side with **argoCD** if you want to deploy your application with **Kubernetes**, and lastly **Prometheus** and **Grafana** for monitoring. 


## Prerequisites

- **Laravel** - Basic laravel and developing application. Basic configuration Cors, Authorization, Middleware and ETC for good practices and development staging.
- **Docker** - Basic Docker to containerized **Laravel** application or any applications. Also you need **Docker Hub** or any **Elastic Container Services** to push your docker images.
- **Nginx** - Basic knowledge of nginx for reverse proxy. 
- **Jenkins** - Basic Jenkins to automate CI/CD. Basic roles, Pipeline Scripts and Concepts of CI/CD.
- **SonarQube** - Basic knowledge of SonarQube and how it works and concepts.
- **Trivy** - Basic knowledge of trivy to scan docker images and files.
- **Kubernetes** - "**Not Required**" but if you want to use **Kubernetes** its a good pracitce along with **ArgoCD**. (*Will Integrate Kubernetes Soon "Laravel Microservices"*).
- **Prometheus and Grafana** - Basic knowledge of this stack for monitoring, concepts and how it works.
- **EC2** - You can use any server of your choice. If you really want to pracitce without paying for the server you only need a Virtual Box that will run your server within your machine. But I recommend that your machine has enough specs and space for the requirements of our **DevSecOps** basic setup. 

## Step by Step Setup

**Phase 1: Servers initial setup**

Step 1: Launch EC2 or create VMware machine if you're using Virtual Machines for the server.
- Create 3 servers, 1 for Laravel application, 1 for Jenkins and SonarQube with Trivy and 1 for Prometheus and Grafana
- Setup each server with fresh and clean OS installation. I recommend you to use Ubuntu Server LTS any version of your choice.
- If your using EC2 allow all TCP ports for SSH create a ssh_key and store it privately into your machine, that's the key for you to SSH your server
- And if you're using VMware click your server one by one, go to settings then network and change the Attached to from *NAT* -> *Bridged Adapter*.

Step 2: SSH to your jenkins/SonarQube server for setup.
- Install Java for Jenkins
```bash
sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version
openjdk version "17.0.8" 2023-07-18
OpenJDK Runtime Environment (build 17.0.8+7-Debian-1deb12u1)
OpenJDK 64-Bit Server VM (build 17.0.8+7-Debian-1deb12u1, mixed mode, sharing)
```
- Check java version and if it is installed
```bash
java --version
```
- Install Jenkins
```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```
- Check jenkins installation
```bash
jenkins --version
```
- Install Docker
```bash
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER  # Replace with your system's username, e.g., 'ubuntu'
newgrp docker
sudo chmod 777 /var/run/docker.sock
```
- Also add jenkins to docker usermod
```bash
sudo usermod -aG docker jenkins
```
- Also your username
```bash
sudo usermod -aG docker $USER
```
- Check Docker if it is installed
```bash
docker --version
```
- Install Docker Compose
```bash
sudo apt install docker-compose
```
- Check docker-compose version
```bash
docker-compose --version
```
- Enable Jenkins
```bash
sudo systemctl enable jenkins
```
- Start Jenkins
```bash
sudo systemctl start jenkins
```
- Check if Jenkins is running
```bash
sudo systemctl enable jenkins
```

Step 3: SSH to your Laravel-App server for setup.
- Install Docker
```bash
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER  # Replace with your system's username, e.g., 'ubuntu'
newgrp docker
sudo chmod 777 /var/run/docker.sock
```
- Also add your username to docker usermod
```bash
sudo usermod -aG docker $USER
```
- Check Docker if it is installed
```bash
docker --version
```
- Install Docker Compose
```bash
sudo apt install docker-compose
```
- Check docker-compose version
```bash
docker-compose --version
```
- Create a folder where you want to clone your application
```bash
mkdir laravel-app
```
- Clone your repo
```bash
cd laravel-app
git clone https://github.com/yourRepo/your-project.git
```
- Copy .env.example
```bash
cp .env.example .env
```
- After copying the env up docker compose
```bash
docker-compose up -d --build
```
- composer install, change you `<service_name>` to you specific service
```bash
docker-compose run <service_name> composer install
```
- Set permission
```bash
sudo chown -R $USER:www-data storage bootstrap/cache
sudo chmod -R 775 storage bootstrap/cache
```
- Then generate key, change you `<service_name>` to you specific service.
```bash
docker-compose run <service_name> php artisan key:generate
```
- after you configure all you can now see your application in port `http://yourip:8000` but your port for your application is depends on your docker configuration.

Step 4: Go and SSH to your jenkins/sonarqube server to configure you jenkins and plugins installation.
- You can access/browse your jenkins server with port 8080 `http://yourip:8080`; make sure that your jenkins is started check the status of your jenkins.
```bash
sudo systemctl status jenkins
```
- In the first glance of the application you can see something like this.

<img title="jenkins first glance/open" alt="jenkins first glance/open" src="https://www.oreilly.com/api/v2/epubs/9781788479356/files/assets/ffea4b1e-fe03-4065-9258-d0159729fa58.png">

- Then go to your terminal to get the initial password that is located in `/var/jenkins_home/secrets/initialAdminPassword` or copy the path that is displayed in your jenkins app.

```bash
sudo cat /var/jenkins_home/secrets/initialAdminPassword # change the path base in your jenskins app.
```
- After just click `continue`.
- Then install the plugins of your choice if you're using jenkins for the first time I recommend to install suggested plugins.

<img title="Plugins installation" alt="Plugins installation" src="https://user-images.githubusercontent.com/75182/41753242-6a9e4ec2-75cc-11e8-9eb7-524b732064d9.png">

- Wait until the installation is done.

<img src="https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWctYXNrLmNzZG4ubmV0L3VwbG9hZC8yMDE2MTIvMTEvMTQ4MTQyMTc3M185NTczNTQucG5n?x-oss-process=image/format,png">

- If the installation will fail just click `retry`.
- Then after just fill up the form for the admin user of your jenkins app.

<img src="https://miro.medium.com/v2/resize:fit:1400/1*CER3iMbg3RxVjVqW1oEbOA.png">

- After click `Click and Continue` and you're will redirected to the dashboard of the jenkins.
- We will install some plugins that we will use in our jenkins.
	- Go to `Manage Jenkins` -> `Plugins` -> `Available`
	- Install the following plugins:
		- `SSH Agent`
		- `Docker `
		- `Docker Pipeline`
	- After click install and restart jenkins.
- Then create a new Job.
	- Click new item or create new job.
	- Enter name of your pipeline then select pipeline then click okay.
	- Scroll down to bottom then change `Pipeline Script` -> `Pipeline script from scm`
	- Choose `scm` -> `git`
	- Paste your github url you want to use or deploy
