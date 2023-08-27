# jenkins-to-ansible-docker-automation

## Tech Stack

<p align="center">
  <img src="https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png" alt="GitHub Logo" width="70" height="70">
  <img src="https://www.jenkins.io/images/logos/jenkins/jenkins.svg" alt="Jenkins Logo" width="60" height="70">
  <img src="https://maven.apache.org/images/maven-logo-black-on-white.png" alt="Maven Logo" width="180" height="50">
  <img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/1f0713b3-9a18-419a-8dc9-69c2a2b9363e" alt="Ansible Logo" width="130" height="90">
  <img src="https://learncybers.com/wp-content/uploads/2023/03/Screenshot-2023-03-18-at-2.58.04-AM.png" alt="Tomcat Logo" width="150" height="80">
  <img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/80447647-d723-42fc-8b60-209d9a511115" alt="Docker Logo" width="100" height="80">
</p>

## Requirements
Before setting up the project, ensure you meet the following prerequisites:

1. **VirtualBox:**
   - Install VirtualBox on your system by downloading it from the official VirtualBox website: [VirtualBox Downloads](https://www.virtualbox.org/wiki/Downloads).
   - Follow the installation instructions for your operating system to complete the setup.

2. **Virtual Machines:**
   - Create five virtual machines with the specified hostnames and purposes as follows:

   | Virtual Machine  | Hostname       | Purpose                                    | Private IP       |
   |------------------|----------------|--------------------------------------------|-----------------|
   | VM1              | jenkins        | Jenkins server for CI/CD pipeline         | 192.168.1.2     |
   | VM2              | tomcat         | Jenkins agent for building and executing jobs | 192.168.1.3               |
   | VM3              | docker         | SonarQube server for code quality analysis | 192.168.1.4     |
   | VM4              | ansible        | Argo CD server for Kubernetes continuous delivery | 192.168.1.5              |

## 1. CI/CD Pipeline with Git , Github , Jenkins and Maven Integration

Target : Automate the Jenkins pipeline to fetch code from GitHub and build it using Maven.
#### Install Jenkins on CentOS
Login to your VM1 
```bash
1. Install java  - yum install java-11-openjdk
Note: Solving the java version selection in case multiple java version exist –>
update-alternatives --config java

2.	sudo wget -O /etc/yum.repos.d/jenkins.repo \
https://pkg.jenkins.io/redhat-stable/jenkins.repo
3.	sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
4.	sudo yum upgrade
5.	sudo yum install jenkins
6.	sudo systemctl daemon-reload

```
You can enable the Jenkins service to start at boot with the command:

```bash
7.	sudo systemctl enable jenkins
```
Use the following command to open port 8080 and reload the firewall service:

```bash
	sudo firewall-cmd --permanent --zone=public --add-port=8080/tcp
	sudo firewall-cmd --reload
```

#### Git and Jenkins Integration
1. Install Git  
```bash
	 yum install git
```
2. Now go to Jenkins UI and install Github plugin

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/a64b0ecd-c570-4a46-ba2a-f8c6f3362d0e" alt="image" width="400" height="300" /> 

3. Go to Global Tool Configuration and setup like this:
Note : Path to Git executable means where is my Git installed. Either we can give full path or just mentioning Git can automatically identify where is Git.

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/ebd03e86-2de2-49d7-8885-734b6a55b965" alt="image" width="360" height="400" /> 

#### Maven and Jenkins Integration
1.	First go to opt folder 
   
   ```bash
    cd /opt
   ```
2.	Install Maven 

   ```bash
   wget https://dlcdn.apache.org/maven/maven-3/3.9.2/binaries/apache-maven-3.9.2-bin.tar.gz
   ```
3.	Now we need to extract the below .tar file 

   <img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/a0563401-b86d-4643-b2c3-bbad9fd23d88" alt="image" width="750" height="90" /> 
  
  ```bash
   Tar -xvzf apache-maven-3.8.3-bin.tar.gz
```

   <img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/87412594-a8d7-4005-92e4-fd0162051ac1" alt="image" width="850" height="60" /> 

4.	Rename the file to make it simple – mv apache-maven-3.8.3 maven
5.	If you go inside the maven bin folder you will see the actual command of Mavne i.e mvn
	
   <img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/ed16098b-2aee-447b-9db0-420219c923b2" alt="image" width="600" height="150" /> 

6.	Now we need to set it up in the environment variable as right now it works only inside the maven bin folder but not outside.

   <img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/ade4cd45-bc3a-4be6-860e-f8c0e26b5c7d" alt="image" width="1000" height="220" /> 

 ```bash
vi ~/.bash_profile
M2_HOME=/opt/maven   (Note : where our apache maven is available)
M2=/opt/maven/bin (Note: binary directory)
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.19.0.7-1.el7_9.x86_64
(Note: where our jdk is available. To find it use – find / -name java-11*)
```
7.	For the values to change either we need to log out and login or we can use – source .bash_profile
8.	Install Maven plugin in Jenkins UI

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/08e76475-477c-4ec0-8ce4-4c3916fed973" alt="image" width="850" height="140" /> 

9. 	Now to integrate Maven to Jenkins go to Global tool configuration and fill the following details for JDK and Maven

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/6de599df-2d27-40ef-8939-45c6bd83bdec" alt="image" width="375" height="350" /> 
<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/f8dc9c59-eaaf-4ddc-be31-805e484023a5" alt="image" width="375" height="375" /> 
 	
####  Build a java project using Jenkins

1.	Create a new job using maven project option 
2.	Fill the Git repo details
   
   <img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/35a1b89c-fcb1-4fd8-9232-728ec8ec0af2" alt="image" width="825" height="300" />
  	
3.	Fill the Maven details
   
   <img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/302057b1-1281-4773-8c6c-2c623e83a5e0" alt="image" width="800" height="200" /> 
   
Note : install means  : install the package into the local repository, for use as a dependency in other projects locally.
4.	Save and apply then build the pipeline .

## 2. Tomcat Integration with CI/CD
Target :

#### Install Tomcat on CentOS
Login to your VM2 

1.	Install Java as did for Jenkins
2.	Go to

      ```bash
  	cd /opt
      ```
      
3.	    
     ```bash
    wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.76/bin/apache-tomcat-9.0.76.tar.gz

    ```
   
4.	Extract the tar.gz file
	
	```bash
    tar -xvzf apache-tomcat-9.0.76.tar.gz
    ```
 
5.	Rename it to tomcat
	
     ```bash
    mv apache-tomcat-9.0.76 tomcat
	 ```
  
6.    ```bash
      cd /tomcat/bin
      ```
      
      and run ll command to see all files

   <img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/0ac50821-d97a-48d3-8769-d88d494237a8" alt="image" width="200" height="350" /> 

7.	Startup.sh is the script we need to start our tomcat services

     ```bash
    ./startup.sh
     ```

8. 	Now if we try to access the Manager App it will give access denied

    <img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/51ac6265-c6ec-498e-856c-2b5ae9359e5a" alt="image" width="1000" height="350" />

    This is because by default we can access the tomcat UI from within the server its hosted but not from outside. In order to access it from outside we need to update the context.xml file.

9.    ```bash
      find / -name context.xml

      ```
   
 <img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/224f10f1-b851-466f-a105-75b96d438dfc" alt="image" width="750" height="100" />

   above command gives 3 context.xml files. comment () Value ClassName field on files which are under webapp directory. After that restart tomcat services to effect these changes

 <img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/8ff6e8bb-c66a-4ad9-b308-580f09249a4e" alt="image" width="500" height="300" />

10.	Update users information in the tomcat-users.xml file goto tomcat home directory and Add below users to conf/tomcat-user.xml file

       ```bash
        <role rolename="manager-gui"/>
	<role rolename="manager-script"/>
	<role rolename="manager-jmx"/>
	<role rolename="manager-status"/>
	<user username="admin" password="admin" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
	<user username="deployer" password="deployer" roles="manager-script"/>
	<user username="tomcat" password="s3cret" roles="manager-gui"/>
        ```
       
11.	create link files for tomcat startup.sh and shutdown.sh so that we start and stop the tomcat services from anywhere

    ```bash
     ln -s /opt/apache-tomcat-8.5.35/bin/startup.sh /usr/local/bin/tomcatup
     ln -s /opt/apache-tomcat-8.5.35/bin/shutdown.sh /usr/local/bin/tomcatdown
    ```
    
12.	Restart your services and now your tomcat App Manager UI will be opening 

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/3221e5e5-61c6-4cce-952e-467d41d85a43" alt="image" width="550" height="300" />

####            Tomcat and Jenkins UI Integration

1.	Go to Jenkins UI -> Manage Jenkins -> Manage Plugins

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/cee6eba0-5474-41bf-8815-1d475bfef516" alt="image" width="500" height="350" />

        Install the above plugins

2.	Now we need to configure tomcat server credentials 
Go to manage Jenkins -> Manage Credentials -> Jenkins -> Global Credentials and click on Add credentials . Fill details as follows:

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/13243e1f-67b6-42db-ac0a-d85b307103b6" alt="image" width="250" height="300" />

Note: we are using “deployer” username because in case of one system  wants to access another system then we need to use managed script roles 

          ```bash
         (<user username="deployer" password="deployer" roles="manager-script"/>)
	 
           ```

3.	Now Integration is Done . Create a new Jenkins job , select the maven project.
4.	Create the job as we created previously . In the end click on the ‘Add Post Build Action’ and select Deploy war/ear to a container.

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/4e58b4e2-a8a9-471f-bb13-5a6001a4f453" alt="image" width="300" height="350" />

5.	Fill the details as follows :

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/eaaeca45-9332-4d69-9d6f-525ba029c708" alt="image" width="200" height="350" />

Note :  
•	WAR/EAR files fields requires the path where our war file is created. Either we can specify the full path or just **/*.war which is good practice
•	In the container choose Tomcat 8

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/a74901ee-d2fa-4bac-8a38-dad42fe0d25f" alt="image" width="200" height="350" />
	
•	In the credentials filed we need to specify the tomcat credentials which we configured earlier.
•	The Tomcat URL fields required the public ip of the server where the tomcat is hosted and the port number.


## 3.	Docker Integration with CI/CD
Target : 

Login to your VM3

#### Install Docker on CentOS


```bash
1.	sudo yum install -y yum-utils
2.	sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
3.	sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
4.	sudo systemctl start docker
5.	sudo docker run hello-world

```

#### Create Tomcat Container

```bash
1.	docker pull tomcat
2.	docker run -d -p 8080  - - tomcat-container tomcat
3.	Access the container at http://VM3_IP:8080
```

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/02c25b87-d9ef-4555-8af9-d0c6e29ad215" alt="image" width="400" height="150" />

4.	To check the issue lets go inside the container –

```bash

Docker exec -it tomcat-container /bin/bash and do ls 

```

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/ab377b8c-be1e-42eb-bfc2-978ce5f0da12" alt="image" width="450" height="125" />

5.	Here if we go inside webapps we will see that its empty and all the required contents are inside webapps.dist

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/d525c6c1-c202-4aca-913e-9bec41e4ae4b" alt="image" width="450" height="125" />

6.	Therefore we need to copy all the contents from webapps.dist to webapps
   
```bash

cp -R * ../webapps/

```

7.	Now if we try to access the Tomcat manager app we can successfully access it.

#### DockerFile for Tomcat Container

1.	Dockerfile to setup tomcat –> 1st Method
   
```bash
FROM centos:7
RUN yum install java -y
RUN mkdir /opt/tomcat
WORKDIR  /opt/tomcat
ADD https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.75/bin/apache-tomcat-9.0.75.tar.gz .
RUN tar -xvzf apache-tomcat-9.0.75.tar.gz
RUN mv apache-tomcat-9.0.75/* /opt/tomcat
EXPOSE 8080
CMD ["/opt/tomcat/bin/catalina.sh","run"]
```

2.	Build Dockerfile

```bash
docker build -t mytomcat  
```

3. 	Run container from image

```bash
docker run -d --name mytomcat-server -p 8083:8080 mytomcat

```

4.	2nd Method is create DockerFile with this content by directly pulling a tomcat image :

```bash
FROM tomcat:latest
RUN cp -R  /usr/local/tomcat/webapps/dist/*  /usr/local/tomcat/webapps

```
 
```bash
5. docker build -t demotomcat
```

```bash
6. docker run -d –name demotomcat-container -p 8084:8080 demotomcat
```

#### Integrate docker with Jenkins

1.	First we need to create a user for docker ,check what all users we have

```bash
cat /etc/passwd
```

2.	Check what all groups are there

```bash
cat /etc/group
```

3.	Create user 
```bash
useradd dockeradmin
Passwd dockeradmin

```

4.	Add this user to docker group 

```bash
usermod -aG docker dockeradmin
```

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/941e17ff-633b-4e91-bbbe-ba1d282be497" alt="image" width="450" height="125" />

5.	If you try to login with this user you wont be able to as by default ec2 don’t allow password based authentication , we have to explicitly enable it.

```bash
vi /etc/ssh/sshd_config and search for /Password
```
uncomment the PasswordAuthentication Yes and comment the PasswordAuthentication no

7.	Save the file and reload the services

```bash
service sshd reload
```

8.   Now Go to Jenkins server and go to the plugins section and install “Publish Over SSH” plugin

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/61c4b5d9-ec0f-4c81-b942-664bcd9e64a1" alt="image" width="330" height="270" />

9.	After installing go to Configure System and click on “Add SSH Server” and add the necessary details

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/0bf3e9dc-590e-4e28-a0b9-090965d83358" alt="image" width="150" height="270" />

    Note : Name – Give any name
            Hostname – IP address of docker server (public or private any)
            Username – Username Jenkins will use to access the server

10.	Next click on advanced and check the “Use password authentication, or use a different key” and specify your dockeradmin password then apply and save.

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/b11b20a7-3ceb-4be1-bebe-f9772359295b" alt="image" width="300" height="200" />

