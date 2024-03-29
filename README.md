# Jenkins_Ansible_on_EC2

Code for the DevOps on AWS [video](https://www.youtube.com/watch?v=oGGi0Gazpe0&list=PLRBkbp6t5gM1leHIU27lzwlSEQqmL3yGH&index=9): Install Jenkins and Ansible on EC2, and deploy Java App on Tomcat  in Docker Containers.

In this video, we'll walk through the process of installing and configuring Jenkins, Ansible and Docker on EC2 Instances, build a .war file from source code in a GitHub Repo using Maven in Jenkins, and deploy to Tomcat in Docker Containers using Jenkins and an Ansible Playbook.

NOTE: You have two paths which you can follow to watch this video.

- Path 1: Start from the beginning and learn how to spin-up a Jenkins Server, then a Tomcat Server, then a Docker Server to deploy Tomcat in a Container, then move on to Ansible and two App Servers (running Docker).
- Path 2: Start at [50:50](https://www.youtube.com/watch?v=oGGi0Gazpe0&list=PLRBkbp6t5gM1leHIU27lzwlSEQqmL3yGH&index=9&t=3050) in the video where we'll build the Jenkins Server, Apps Servers and Ansible Server from scratch.

<hr />

## Commands - Path 1
### Install Jenkins on EC2
```
sudo su -
hostname JenkinsServer
sudo su -

apt update -y

apt install openjdk-11-jdk -y

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
 /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [trusted=yes] \
 https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
 /etc/apt/sources.list.d/jenkins.list > /dev/null

apt-get update -y

apt-get install jenkins -y

systemctl enable jenkins
systemctl start jenkins
systemctl status jenkins

apt install maven -y
echo "export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64" >> ~/.bashrc
echo "export MAVEN_HOME=/usr/share/maven" >> ~/.bashrc

apt install git -y
```


### Configure Jenkins
```
cat /var/lib/jenkins/secrets/initialAdminPassword

JAVA_HOME: /usr/lib/jvm/java-11-openjdk-amd64
git: /usr/bin/git
MAVEN_HOME: /usr/share/maven
```


### Install Tomcat on EC2
```
sudo su -
hostname TomcatServer
sudo su -

yum update -y

yum install java-1.8* -y
$(dirname $(dirname $(readlink -f $(which javac))))

cd ~
nano .bash_profile
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.372.b07-1.amzn2.0.1.x86_64
PATH=$PATH:$HOME/bin:$JAVA_HOME


cd /opt
wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.76/bin/apache-tomcat-9.0.76.tar.gz
tar -xvzf apache-tomcat-9.0.76.tar.gz
mv apache-tomcat-9.0.76 tomcat

cd ..
find -name context.xml

cd /opt/tomcat
nano conf/tomcat-users.xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="manager-jmx"/>
<role rolename="manager-status"/>
<user username="admin" password="admin" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
<user username="deployer" password="deployer" roles="manager-script"/>
<user username="tomcat" password="tomcat" roles="manager-gui"/>

cd bin/
./startup.sh
```


### Install Docker on EC2
```
sudo su -
hostname DockerServer
sudo su -

yum update -y
yum install docker -y

service docker start
service docker status

docker pull tomcat:8.0
docker image ls

docker run -d --name tomcat -p 8080:8080 tomcat:8.0
docker ps
```


### Configure Docker
```
sudo su -
useradd docker-admin
passwd docker-admin

usermod -aG docker docker-admin
id docker-admin

Enable Password Auth
cd /etc/ssh
nano sshd_config
systemctl reload sshd


cd /home/docker-admin
su - docker-admin
nano Dockerfile


Exec Command
cd /home/docker-admin; docker build -t webapp-img .; docker run -d --name webapp -p 8080:8080 webapp-img;
```


### Install Ansible on EC2
#### Ansible Server
```
sudo su -
hostname AnsibleServer
sudo su -

yum update -y
amazon-linux-extras install ansible2 -y

ansible --version

ssh-keygen -t rsa
cat .ssh/id_rsa.pub 
```

#### AppServer 1 & AppServer2
```
sudo su -
hostname AppServer1
sudo su -

nano .ssh/authorized_keys
```

#### Ansible Server
```
cd /etc/ansible

nano ansible.cfg 
mv hosts hosts.BKUP
nano hosts

ansible appservers -m ping
```


#### AppServer 1 & AppServer2
```
yum update -y
yum install docker -y

service docker start
service docker status

useradd docker-admin
passwd docker-admin

usermod -aG docker docker-admin
id docker-admin

cd /etc/ssh
nano sshd_config
systemctl reload sshd

cd /home/docker-admin
su - docker-admin
nano Dockerfile
```

#### Ansible Server
```
mkdir playbooks && cd playbooks
nano playbook.yml

cd ..
ansible-playbook playbooks/playbook.yml -i hosts --check
ansible-playbook playbooks/playbook.yml -i hosts 

useradd ansible-admin
passwd ansible-admin

cd /etc/ssh
nano sshd_config
systemctl reload sshd

Command:  cd /etc/ansible; ansible-playbook playbooks/playbook.yml -i hosts
```

<hr />

## Commands - Path 2
### Install Jenkins on EC2
```
sudo su -
hostname JenkinsServer
sudo su -

apt update -y

apt install openjdk-11-jdk -y

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
 /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [trusted=yes] \
 https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
 /etc/apt/sources.list.d/jenkins.list > /dev/null

apt-get update -y

apt-get install jenkins -y

systemctl enable jenkins
systemctl start jenkins
systemctl status jenkins

apt install maven -y
echo "export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64" >> ~/.bashrc
echo "export MAVEN_HOME=/usr/share/maven" >> ~/.bashrc

apt install git -y
```


### Configure Jenkins
```
cat /var/lib/jenkins/secrets/initialAdminPassword

JAVA_HOME: /usr/lib/jvm/java-11-openjdk-amd64
git: /usr/bin/git
MAVEN_HOME: /usr/share/maven
```


## AppServer1 & AppServer2
### Install Docker
```
sudo su -
hostname AppServer1
sudo su -

yum update -y
yum install docker -y

service docker start
service docker status
```

### Configure Docker
```
useradd docker-admin
passwd docker-admin

usermod -aG docker docker-admin
id docker-admin


Enable Password Auth
cd /etc/ssh
nano sshd_config
systemctl reload sshd


cd /opt
mkdir docker
chown -R docker-admin:docker-admin docker
cd docker
nano Dockerfile
From tomcat:8-jre8 
COPY ./webapp.war /usr/local/tomcat/webapps
chown -R docker-admin:docker-admin /opt/docker


Docker - configure Docker Server
New Item
  SourceFiles: webapp/target/*.war
  Remove Prefix: webapp/target/
  Remote Dir: //opt//docker

Build
ls -al /opt/docker

Exec Command
cd /opt/docker; docker build -t webapp-img .; docker run -d --name webapp -p 8080:8080 webapp-img;

Build
ls -al
docker image ls
docker ps
http://PUBLIC_IP:8080/webapp/
```


### Install Ansible on EC2
#### Ansible Server
```
sudo su -
hostname AnsibleServer
sudo su -


useradd ansible-admin
passwd ansible-admin
visudo
ansible-admin ALL=(ALL)       NOPASSWD:ALL


Enable Password Auth
cd /etc/ssh
nano sshd_config
systemctl reload sshd

sudo su ansible-admin
cd ~

ssh-keygen -t rsa


sudo su -
yum update -y
amazon-linux-extras install ansible2 -y
```


## AppServers
```
useradd ansible-admin
passwd ansible-admin
visudo
ansible-admin ALL=(ALL)       NOPASSWD:ALL
```

## Ansible Server
```
cd /etc/ansible

?????nano ansible.cfg 
mv hosts hosts.BKUP
nano hosts


sudo su - ansible-admin
ssh-copy-id PRIVATE_IP

cd .. (home)
ansible appservers -m ping

exit
cd /etc/ansible
mkdir playbooks
chown -R ansible-admin:ansible-admin playbooks
cd playbooks
nano playbook.yml

chown -R ansible-admin:ansible-admin /etc/ansible/playbooks

cd ..
sudo su ansible-admin
ansible-playbook playbooks/playbook.yml -i hosts --check
ansible-playbook playbooks/playbook.yml -i hosts 
```