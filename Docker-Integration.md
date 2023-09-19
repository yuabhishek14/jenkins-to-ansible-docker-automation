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
2.	docker run -d -p 8081:8080  --name tomcat-container tomcat
3.	Access the container at http://VM3_IP:8080
```

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/02c25b87-d9ef-4555-8af9-d0c6e29ad215" alt="image" width="580" height="150" />

4.	To check the issue lets go inside the container –

```bash

Docker exec -it tomcat-container /bin/bash and do ls 

```

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/ab377b8c-be1e-42eb-bfc2-978ce5f0da12" alt="image" width="610" height="90" />

5.	Here if we go inside webapps we will see that its empty and all the required contents are inside webapps.dist

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/d525c6c1-c202-4aca-913e-9bec41e4ae4b" alt="image" width="600" height="90" />

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
RUN cp -R  /usr/local/tomcat/webapps.dist/*  /usr/local/tomcat/webapps

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

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/941e17ff-633b-4e91-bbbe-ba1d282be497" alt="image" width="510" height="90" />

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

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/61c4b5d9-ec0f-4c81-b942-664bcd9e64a1" alt="image" width="330" height="250" />

9.	After installing go to Configure System and click on “Add SSH Server” and add the necessary details

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/0bf3e9dc-590e-4e28-a0b9-090965d83358" alt="image" width="150" height="270" />

    Note : Name – Give any name
            Hostname – IP address of docker server (public or private any)
            Username – Username Jenkins will use to access the server

10.	Next click on advanced and check the “Use password authentication, or use a different key” and specify your dockeradmin password then apply and save.

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/b11b20a7-3ceb-4be1-bebe-f9772359295b" alt="image" width="300" height="200" />

11.	Now in the Dockerhost server lets create a folder where we will copy our artifacts 

```bash
cd /opt
```

Here we will create a folder 

```bash
mkdir docker
```
Give permission to dockeradmin user  

```bash
chown -R dockeradmin:dockeradmin docker
```

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/4ef6a612-0544-4d38-9efe-d272b482b94a" alt="image" width="530" height="80" />


Also we have to copy the DockerFile to the docker folder and give permission 

```bash
chown -R dockeradmin:dockeradmin /opt/docker
```
<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/8418122e-aac5-487e-ba69-7dd3f1f137ec" alt="image" width="530" height="80" />

Also we need to update our DockerFile to copy our artifact to the container /usr/local/tomcat/webapps folder.
Add : 
```bash
copy ./*.war  /usr/local/tomcat/webapps
```

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/a380f3c6-d26a-4f52-bac6-23687585c8e4" alt="image" width="540" height="80" />

12. Now create a Jenkins job as we created previously and in the "Post Build Actions" select "Send build artifacts over SSH" and fill all the fields:

   - **Name**: Select the dockerhost server
   - **Sources files**: Place where the artifact is present
   - **Remove prefix**: Part of the source path which you want to remove while copying to the target path.
   - **Remote Directory**: Target path

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/25706346-4995-42be-be7a-64364fe82e79" alt="image" width="400" height="400" />
<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/7c045de0-12f7-42c9-8c92-63e059c60a49" alt="image" width="210" height="400" />

13. In the Exec command section add the docker commands to create a tomcat container to which we will deploy our artifacts

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/f9995a91-e9f0-40d4-9c02-56fa31f20edc" alt="image" width="500" height="180" />