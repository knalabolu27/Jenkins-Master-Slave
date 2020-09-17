# Jenkins-Master-Slave
# Build and Deploy Java Maven HelloWorld Application in Tomcat 9 with JDK 8 in Amazon EC2

## Launch an instance
Select amazon linux instance and configure the settings. In the step-5 Add tags with the name and in the step-6 configure security group with a new security group(open port 22 and 8080 in the inbound rule with CIDR 0.0.0.0/0 to access from anywhere for now).
A key pair gets downloaded before launching the instance
### Change the read only permissions to the pem file
`chmod 400 Work.pem`
### Connect to the instance from the command line
`ssh -i "<path of the pem file>" ec2-user@ec2-36-215-39-147.us-west-2.compute.amazonaws.com`

## Update all the applications before installing
`sudo yum update -y`

## Install java
https://bhargavamin.com/how-to-do/setting-up-java-environment-variable-on-ec2/
```
sudo yum install java-1.8.0
sudo yum remove java-1.7.0-openjdk
```
## To check the java installed location
`file $(which java)`
### To set the java path
```
sudo /usr/sbin/alternatives --set java /usr/lib/jvm/jre-1.8.0-openjdk.x86_64/bin/java
sudo /usr/sbin/alternatives --set javac /usr/lib/jvm/jre-1.8.0-openjdk.x86_64/bin/javac
export JAVA_HOME="/usr/lib/jvm/jre-1.8.0-openjdk.x86_64"
export PATH=$JAVA_HOME/bin:$PATH  
```
### To set the java path to bash_profile
add below content to `~/.bash_profile` file
```
export JAVA_HOME="/usr/lib/jvm/jre-1.8.0-openjdk.x86_64"
export PATH=$JAVA_HOME/bin:$PATH 
```
### To check JAVA_HOME
`echo $JAVA_HOME`


## use above steps to install java on all the master and slave nodes

## Install Jenkins on master node  
https://medium.com/@itsmattburgess/installing-jenkins-on-amazon-linux-16aaa02c369c
```
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
sudo rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key
sudo yum install jenkins
sudo service jenkins start
```
### Jenkins to automatically start when your instance is started
`sudo chkconfig --add jenkins`
### To access Jenkins server
Make sure port 8080 is open in security group inbound rules of that virtual machine.
`http://{ec2-public-dns}:8080`
### Jenkins Home directory
`/var/lib/jenkins/`
#### Configure Jenkins
- The default Username is `admin`
- Grab the default password 
  - Password Location:`/var/lib/jenkins/secrets/initialAdminPassword`
- `Skip` Plugin Installation; _We can do it later_
- Change admin password
  - `Admin` > `Configure` > `Password`
- Configure `java` path
  - `Manage Jenkins` > `Global Tool Configuration` > `JDK`  
- Create another admin user id

# Configure Jenkins Slaves on AWS EC2
Jenkins is a self-contained Java-based program, ready to run out-of-the-box, with packages for Windows, Mac OS X and other Unix-like operating systems. As an extensible automation server, Jenkins can be used as a simple CI server or turned into the continuous delivery hub for any project.

### Follow this article in **[Youtube](https://youtu.be/hwrYURP4O2k)**
![Jenkins Master and Slave Configuration](https://raw.githubusercontent.com/miztiik/DevOps-Demos/master/setup-jenkins-slave/images/Jenkins%20Master%20and%20Slave%20Configuration.png)

### Prerequisites
1. Jenkins Master Running [Get help here](https://youtu.be/-0dkiteJEuE)
1. EC2 RHEL 7.x Instance - _for Slave Node_ [Get help here](https://www.youtube.com/watch?v=KDtS6BzJo3A)
   - With Internet Access
   - Security Group with Port `8080` open for internet
   - Java v1.8.x 

## Install Java
We will be using open java for our demo, Get latest version from http://openjdk.java.net/install/. Also configure the default `JAVA_HOME` path
```sh
yum install java-1.8*
#yum -y install java-1.8.0-openjdk
```
## Setup Jenkins Slave
```sh
# Create user and add the user to wheel group
useradd jenkins-slave-01
# Create SSH Keys
sudo su - jenkins-slave-01
ssh-keygen -t rsa -N "" -f /home/jenkins-slave-01/.ssh/id_rsa
# The private and public keys will be created at these locations `/home/jenkins-slave-01/.ssh/id_rsa` and `/home/jenkins-slave-01/.ssh/id_rsa.pub`
cd .ssh
cat id_rsa.pub > authorized_keys
chmod 700 authorized_keys
```

### Configuration on Master
Copy the slave node's public key[`id_rsa.pub`] to Master Node's `known_hosts` file
```sh
mkdir -p /var/lib/jenkins/.ssh
cd /var/lib/jenkins/.ssh
ssh-keyscan -H SLAVE-NODE-IP-OR-HOSTNAME >>/var/lib/jenkins/.ssh/known_hosts
# ssh-keyscan -H 172.31.38.42 >>/var/lib/jenkins/.ssh/known_hosts
chown jenkins:jenkins known_hosts
chmod 700 known_hosts
```

#### Configure the Slave using `Manage Jenkins`
Configure the node as shown here
`Manage Jenkins` > `Manage Nodes` > `New Node`
![Jenkins Master and Slave Configuration](https://raw.githubusercontent.com/miztiik/DevOps-Demos/master/setup-jenkins-slave/images/Slave-Node-Configuration-01.png)
### Test Jenkins Jobs
1. Create “new item”
1. Enter an item name – `My-First-Project`
   - Chose `Freestyle` project
1. Under `General` Section
   - Choose `Restrict where this project can be run`
     - Update your _jenkins slave label_ `Java` 
1. Under Build section
   Execute shell
   ```sh
   #!/bin/bash
   echo "_______________________________"
   echo "|                             |"
   echo "|   Welcome to Valaxy Demo    |"
   echo "|           _nnnn_            |"
   echo "|          dGGGGMMb           |"
   echo "|         @p~qp~~qMb          |"
   echo "|         M|@||@) M|          |"
   echo "|         @,----.JM|          |"
   echo "|        JS^\__/  qKL         |"
   echo "|       dZP        qKRb       |"
   echo "|      dZP          qKKb      |"
   echo "|     fZP            SMMb     |"
   echo "|     HZM            MMMM     |"
   echo "|     FqM            MMMM     |"
   echo "|   __| '.        |\dS'qML    |"
   echo "|   |    '.       | ' \Zq     |"
   echo "|  _)      \.___.,|     .'    |"
   echo "|  \____   )MMMMMP|   .'      |"
   echo "|       '-'       '--' hjm    |"
   echo "_______________________________"
   ```
1. Save your job 
1. Build job
1. Check "console output"

