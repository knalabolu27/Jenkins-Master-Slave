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
