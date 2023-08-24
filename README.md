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
   | VM1              | jenkinsMachine | Jenkins server for CI/CD pipeline         | 192.168.1.2     |
   | VM2              | tomcat   | Jenkins agent for building and executing jobs | 192.168.1.3               |
   | VM3              | docker         | SonarQube server for code quality analysis | 192.168.1.4     |
   | VM4              | ansible        | Argo CD server for Kubernetes continuous delivery | 192.168.1.5              |

## 1. CI/CD Pipeline with Git , Github , Jenkins and Maven Integration

Target : Automate the Jenkins pipeline to fetch code from GitHub and build it using Maven.
#### Install Jenkins on CentOS
