## 4.	Ansible  Integration with CI/CD
**Target** : Automate the Jenkins pipeline to fetch code from GitHub, build it using Maven, and transfer artifacts to an Ansible server. Ansible will utilize a Dockerfile to create a Docker image, which will be committed to Docker Hub. When executing an Ansible playbook, the Docker host will pull the image from Docker Hub and deploy it as a container.

#### Configure Ansible Machine
On the VM4 execute these steps :

1. 
```bash
sudo yum install -y yum-utils
```

2. Add new user
   
```bash
useradd ansadmin
passwd ansadmin
```

3. Add this user to sudos file
   
```bash
visudo 
Then go to the end of file by shift+G and add :
Ansadmin ALL=(ALL)  NOPASSWD : ALL and save the file
```

4. Now we need to enable password based authentication for that
   
```bash
vi /etc/ssh/ssh_config 
```
in this file uncomment the PasswordAuthentication yes and comment the PasswordAuthentication No

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/b808521e-7f44-4854-9e97-58f91ec92f38" alt="image" width="540" height="120" />

Then reload the ssh service

```bash
service sshd reload
```

5.	Now we need to create keys for ansadmin user

```bash
sudo su – ansadmin
ssh-keygen
```

6.	Now we have found our private key at  **/home/ansadmin/.ssh/id_rsa** and public key at **/home/ansadmin/.ssh/id_rsa.pub**


7.	Install Ansible 

```bash
sudo su –
sudo yum install epel-release
sudo yum install ansible
```

#### Ansible – Docker Integration
Go to Docker host (VM3) and execute these commands :

1.	
```bash
sudo su –
```
2.	
```bash
useradd ansadmin
passwd ansadmin
```
3.	Add this user to sudos file
```bash
visudo 
Then go to the end of file by shift+G and :
Ansadmin ALL=(ALL)  NOPASSWD : ALL and save the file 
```
4.	Check if password based authentication is enabled or not

```bash
grep Password /etc/ssh/ssh_config
```
**Switch to Ansible host**
1.	Here we need to add our docker host as managed node (meaning it is managed by ansible)
This we can give in the default inventory – vi /etc/ansible/hosts
Delete the entire content inside the file and add the docker host ip address and save the file

2.	Now we need to copy our ansadmin user keys on to target ansadmin user account

```bash
sudo su – ansadmin
ssh-copy-id docker_hostip
```

3.	Now check if we are able to connect to our managed system which is docker host
```bash
ansible all -m ping
```

#### Ansible – Jenkins Integration

Go to Jenkins UI >> Manage Jenkins >>  Configuration System

1.	In the “Publish over SSH” section  add the ansible host details

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/6c6a70f4-6d30-4697-9418-1de6b372e508" alt="image" width="200" height="390" />

In the advanced section add the ansadmin password . Click on the test configuration to test connectivity .

2.	Create a Jenkins job named “Copy_Artifacts_onto_Ansible”by using previously created docker job .
Remove the entire “Exec Command” section 

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/034f932d-82c6-4efd-98da-72ed3d2c9e16" alt="image" width="300" height="360" />

We will create a ansible playbook to perform the same activity.

3.	Now since //opt//docker doesn’t exist on ansible host we need to create it

```bash
cd /opt
mkdir docker 
sudo chown ansadmin:ansadmin docker
```

#### Build Image and container on Ansible

1.	Install docker on ansible host

```bash
sudo yum install docker -y
```
2.	Add you ansadmin to docker group 

```bash
sudo usermod -aG docker ansadmin
```
3.	Start docker service 

```bash
service docker start
```
4.	Create docker file

```bash
vi Dockerfile

FROM tomcat:latest
RUN cp -R  /usr/local/tomcat/webapps.dist/*  /usr/local/tomcat/webapps
COPY ./*.war  /usr/local/tomcat/webapps
```
5.	Try to manually create a docker image and run container

```bash
Docker build -t regapp:v1
If you get error for path /var/run/docker.sock then give permission
sudo chmod 777 /var/run/docker.sock
```
Now create a container 

```bash
docker run -d –name regapp-server -p 8081:8080 regapp:v1
```

#### Ansible Playbook to create Image and Container

1.	Login with ansadmin
2.	Update your inventory file , go to /etc/ansible/hosts
```bash
[dockerhost]
Docker machine ip (VM3 IP)

[ansible]
Ansible machine ip (VM4 IP)
```

3.	Now if we try 
```bash
ansible all -a uptime 
```
you will observe that it will be able to connect to dockerhost only not to ansible host since earlier we have copied our ssh keys to dockerhost now we need to copy our ssh key to our own system that is ansible host

```bash
ssh-copy-id ansible_ip or you can do ssh-copy-id localhost
```

4.	Inside our /opt/docker folder we have Dockerfile and webapp.war file , now we will create ansible playbook

```bash
vi regapp.yml
```
```bash
---

- hosts: ansible
  tasks:
  - name: create docker image
    command: docker build -t regapp:latest .
    args:
     chdir: /opt/docker
```     
5.	Now add steps to add tag to the image and push the image to docker hub

```bash
vi regapp.yml
```

```bash
---
- hosts: ansible
  tasks:
  - name: create docker image
    command: docker build -t regapp:latest .
    args:
     chdir: /opt/docker
  - name: create tag to push image onto dockerhub
           command: docker tag regapp:latest abhishekdevops14/regapp:latest
  - name: push docker image
    command: docker push abhishekdevops14/regapp:latest

```    

validate the playbook by  

```bash
ansible-playbook regapp.yml --check
```
#### Final Jenkins Job

1.	Open the jenkins job **“Copy_Artifacts_onto_Ansible”** and in the **“Exec Command”** section add the command to execute the ansible playbook to create a docker image and push it to docker hub

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/70f9f1db-e2ca-4bf9-bee0-42cfaee0548c" alt="image" width="300" height="370" />

2.	Now we need to create another ansible playbook to get the docker image from the docker hub and create a container on dockerhost machine
Login to your ansible machine and go to the /opt/docker folder

```bash
vi deploy_regapp.yml
```

```bash
---
- hosts: dockerhost
  tasks:
  - name: stop existing container
    command: docker stop regapp-server
    ignore_errors: yes

  - name: remove the container
    command: docker rm regapp-server
    ignore_errors: yes

  - name: remove image
    command: docker rmi abhishekdevops14/regapp:latest
    ignore_errors: yes

  - name: create container
    command: docker run -d –name regapp-server -p 8082:8080 abhishekdevops14/regapp:latest

```

3.	Give permission to Sudo chmod 777 /var/run/docker.sock on dockerhost
4.	Now in the job add the command to execute the deploy_regapp.yml

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/d8246d12-abca-4101-aedf-70da5fa5f82c" alt="image" width="300" height="370" />