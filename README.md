# jenkins-docker-agent


this repo contains a Dockerfile for creating a custom Jenkins docker slave.
you need to download the code and perform a docker build.


launch a server (ubuntu 15Gib and open port 8080 on SG) or pull a jenkins image
connect to the instance and update and edit the /etc/hostname file to Jenkins-Master (the name given to instance on launch)
reboot system
### update the system and install jenkins
sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version
#openjdk version "17.0.8" 2023-07-18
#OpenJDK Runtime Environment (build 17.0.8+7-Debian-1deb12u1)
#OpenJDK 64-Bit Server VM (build 17.0.8+7-Debian-1deb12u1, mixed mode, sharing)
#jenkins
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
# incase there is a problem with the file not existing and you tried using bash and you had the same issue, try reloading the systemctly daemon
sudo systemctl daemon-reload
launch another ubuntu instance 
update and upgrade then install jenkins starting with java 
runtime
sudo bash -c "#!bin/bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins"
## for performance and security reasons it is good to always use a controller and agents to run your jobs in jenkins. In this project, we will use a virtual machine and run the agents on docker connecting the agents to the controller via SSH
To run this guide you will need a machine with:
Java installation
Jenkins installation
Docker installation
SSH key pair
on either of the machines, we will generate the SSH key. I will use the Jenkins-master server.
   ssh-keygen -f ~/.ssh/jenkins_agent_key
then go to the jenkins dashboard >>>>manage jenkins>>>>manage credentials
from global items select Add credentials
Fill in the form:
Kind: SSH Username with the private key;
id: jenkins
description: The Jenkins SSH key
username: jenkins
Private Key: select Enter directly and press the Add button to insert the content of your private key file at ~/.ssh/jenkins_agent_key. if you don't get access go to the directory directly
cd .ssh 
ls
cat jenkins_agent_key
copy contents of the private key and paste
Passphrase: fill in your passphrase used to generate the SSH key pair (leave empty if you didn’t use one in the previous step) and then press the create button.
### Docker container as Jenkins Build agents
 # SETUP DOCKR REMOTE API
 # INSTALL AND CONFIGURE DOCKER PLUGIN
 # CREATE DOCKER AGENT CLOUD
 # VALIDATE WITH A JOB

for the agent, we will set docker up in another instance and then open a port range and configure the docker host as a remote API
build the docker agent image
configure jenkins to integrate with Jenkins-master
configure docker agent templates
run a freestyle job to test
on Amazon Linux machine
  sudo yum update -y
 sudo yum install docker -y
 sudo systemctl start docker
 sudo docker run hello-world (verify download is successful)
 sudo systemctl enable docker
 docker --version / sudo systmectl status docker
 you can add user to docker group (optional)
sudo usermod -a -G docker $(whoami)
to expose docker API on port 4243
 sudo vi /lib/systemd/system/docker.service
 comment ExecStart... line 
 (ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock $OPTIONS $DOCKER_STORAGE_OPTIONS $DOCKER_ADD_RUNTIMES)
 and add line below
  ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock
  ## save configuration changes restart docker service
  sudo systemctl daemon-reload
  sudo service docker restart
  ## ensure that everything is fine
  sudo systemctl status docker
  verify that docker host can be accessed on port 4243 with curl command
    curl http://localhost:4243/version (use publiciIP in the place of localhost)
   returns; 
  {"Platform":{"Name":""},"Components":[{"Name":"Engine","Version":"20.10.25","Details":{"ApiVersion":"1.41","Arch":"amd64","BuildTime":"2023-12-29T20:38:05.000000000+00:00","Experimental":"false","GitCommit":"5df983c","GoVersion":"go1.20.12","KernelVersion":"5.10.205-195.807.amzn2.x86_64","MinAPIVersion":"1.12","Os":"linux"}},{"Name":"containerd","Version":"1.7.11","Details":{"GitCommit":"64b8a811b07ba6288238eefc14d898ee0b5b99ba"}},{"Name":"runc","Version":"1.1.11","Details":{"GitCommit":"4bccb38cc9cf198d52bebf2b3a90cd14e7af8c06"}},{"Name":"docker-init","Version":"0.19.0","Details":{"GitCommit":"de40ad0"}}],"Version":"20.10.25","ApiVersion":"1.41","MinAPIVersion":"1.12","GitCommit":"5df983c","GoVersion":"go1.20.12","Os":"linux","Arch":"amd64","KernelVersion":"5.10.205-195.807.amzn2.x86_64","BuildTime":"2023-12-29T20:38:05.000000000+00:00"}
  ## install git in the docker machine so you can pull the code for the image we will build for the agent so that we can create as many containers we want to work as agents for as many jobs that we want.
  sudo yum install git 
  clone the repo  https://github.com/MesangaE/jenkins-docker-agent.git
  ### Build a docker image
cd into jenkins-docker-slave (modify and change the name)
sudo docker build -t my-jenkins-agent .
sudo docker images
  ### login to jenkins dashboard to configure jenkins with docker plugins
  go to manage jenkins>>manage plugins>>>available plugins>>>docker>>install
  manage jenkins>>manage nodes and clouds>>configure cloud>>add new cloud>>choose docker
  for docker, cloud details put DNS name of the virtual machine carrying docker using tcp protocol
  tcp://ec2-50.17.32.158.compute-1.amazonaws.com:4243
  you do not need to add any secret credentials
  enable and test connection
  ### Add docker templates
  enter template name in this case docker for consistency
  enable it
  enter the name of the image we created
  remote file system root: /home/jenkins
  keep it at 'use this node as much as possible'
  name and labels: docker-agent
  connect method in this case is SSH but you can also go with the JNLP method
  ## Note that the .ssh directory with a key that you generated should be in the repo
  we will go with a 'non-verifying verification strategy'
  pull strategy: never pull (since we are already working with our image that will provision agents as needed.)
  apply and save
  run a freestyle job to see!!
