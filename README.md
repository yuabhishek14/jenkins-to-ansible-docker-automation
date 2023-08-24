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

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/a64b0ecd-c570-4a46-ba2a-f8c6f3362d0e" alt="image" width="360" height="400" /> 

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
	
<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/ed16098b-2aee-447b-9db0-420219c923b2" alt="image" width="800" height="200" /> 

