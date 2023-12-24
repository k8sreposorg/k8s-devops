# DevStack

# Continuous Integration - Conitinuous Delivery - Continuous Deployment

# Step 1: Launch 3 instances with names  jenkins, artifactory and tomcat and open all traffic in security group

Note down public ips

jenkins -

artifactory -

tomcat - 


# Step 2: Install jenkins server on the above jenkins instance

$sudo yum install java-1.8.0-openjdk-devel

$curl --silent --location http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo | sudo tee /etc/yum.repos.d/jenkins.repo

$sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

$sudo yum install jenkins

$sudo systemctl start jenkins

$sudo systemctl enable jenkins

Access jenkins server:  http://<PUBLIC_IP>:8080


# Step 3: Install Artifactory server on the above attifactory instance

$yum install -y java-1.8.0-openjdk-devel

$sudo chmod 777 /etc/profile

$sudo echo "export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-1.el7_6.x86_64" >> /etc/profile

$source /etc/profile

$env | grep JAVA

$yum install -y net-tools rsync

$wget https://bintray.com/jfrog/artifactory-rpms/rpm -O bintray-jfrog-artifactory-rpms.repo

$sudo mv bintray-jfrog-artifactory-rpms.repo /etc/yum.repos.d/

$ sudo yum install jfrog-artifactory-oss

$sudo echo "export ARTIFACTORY_HOME=/opt/jfrog/artifactory" >> /etc/profile

$source /etc/profile

$env | grep ARTIFACTORY_HOME

$systemctl start artifactory

$systemctl enable artifactory

$systemctl status artifactory

acess artifactory  server  http://<public ip>:8081



# Step 4: Install tomcat server on the above tomcat instance

$sudo yum update -y && sudo yum install wget -y && sudo yum install git -y

Step 2: install openjdk

sudo yum install java-1.8.0-openjdk-devel -y

Step 3: install set up tomcat app server

$wget https://downloads.apache.org/tomcat/tomcat-8/v8.5.64/bin/apache-tomcat-8.5.64.tar.gz

$mkdir tomcat

$sudo tar xvf apache-tomcat-8*tar.gz -C /home/centos/tomcat --strip-components=1

$sudo chown -R centos:centos /home/centos/tomcat/*

$sudo sh /home/centos/tomcat/bin/startup.sh



# Step 5 :Set up Jenkins Job to fetch repo and to build the code and to make war package

New Item => Enter an Item name and choose free style project and submit ok 

1.General => Description: this is my first job


3.Source code management


  Git => respositories => Repository URL: https://github.com/cloudstones/DevOps.git
  
  branch to build : main
  
4.Build triggers: optional

6.Build environment(optional)
 
 7.Build => Invoke top level maven targets => Goals: package

# Step 6 :Set up Jenkins Job to fetch repo and to build the code and to make war package and release war file to Artifactory server

=>creating repo

Login to Jfrog Artifactory server=> Administration => Repositories => Local => ADD repositories => local repository => Basic => Package type : maven => Repository key => myrepo=> click on SAVE and FINISH

=>install artifactory plugin in Jenkins

Manage Jenkins=>manage plugins => Available => Artifactory

=>add ertifactory server details in jenkins

manage jenkins=> configure system => artifactory=> add artifactory server

server id jfrog

url : https://34.7.1.28:8081/artifactory

username: admin

pwd: admin_123

apply and save

step 3 to create Jenkins Job

New item=> create free style project=> general=> scm=> build triggers=> build environment

choose maven3-artifactory integration

artifactory configuration'

artifactory server: https://34.7.1.28:8081/artifactory

target release repository: <target repo>
  
  
=> Build

choose Invoke artifactory maven3

select goal

package

Note
------
if you get any error as below 

This build step needs to know where your Maven3 is installed. Please do so from

Then go to Global tool configuration => choose Maven


# Step 7 :Set up Jenkins Job to fetch repo and to build the code and to make war package and release war file to Artifactory server and to deploy Tomcat server

Step 1: Configuration chnages on Tomcat server

=>vi /opt/tomcat/conf/tomcat-users.xml

<role rolename="manager-script"/>

<role rolename="manager-gui"/>

<role rolename="admin-gui"/>

<user username="admin" password="admin_123" roles="manager-gui"/>

<user username="admin" password="admin_123" roles="admin-gui"/>

 <user username="admin" password="admin_123" roles="manager-script" />

=>vi /opt/tomcat/webapps/manager/META-INF/context.xml 

<!--  <Valve className="org.apache.catalina.valves.RemoteAddrValve"

         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->

 
manage jenkins => manage plugins => available  =>  Deploy to conatiner

post build actions => Deploy war/ear to a container => WAR/EAR files: **/*.war =>Context path: optional => Containers - add container: Tomcat 8x => Tomcat url: http:<PUBLIC_IP/8080




# OpsStack

# Step 1: Build AMI using Packer

$git clone https://github.com/cloudstones/k8s-packer.git

$cd k8s-packer/src

$make


# Step 2: Build EKS cluster using terraform

$git clone https://github.com/cloudstones/k8s-terraform.git

$cd k8s-terratform/src

chnage custome AMI in worker.tf file

$vi modules/containers/eks/workers.tf

image_id =""
key_name =""

$vi config.json

"myregion" : "",

$terraform init .

$terraform validate -var-file=config.json .

$terraform apply -var-file=config.json .

configure kubectl

$aws eks --region $(terraform output -raw region) update-kubeconfig --name $(terraform output -raw cluster_name)


# Step 3: Deploy kubernetes manifest files using helmfile tool

$git clone https://github.com/cloudstones/k8s-helm.git

cd k8s-helm/src

helmfile sync
