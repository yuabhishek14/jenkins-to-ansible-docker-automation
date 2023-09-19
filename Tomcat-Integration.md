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

   <img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/0ac50821-d97a-48d3-8769-d88d494237a8" alt="image" width="200" height="300" /> 

7.	Startup.sh is the script we need to start our tomcat services

     ```bash
    ./startup.sh
     ```

8. 	Now if we try to access the Manager App it will give access denied

    <img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/51ac6265-c6ec-498e-856c-2b5ae9359e5a" alt="image" width="1050" height="320" />

    This is because by default we can access the tomcat UI from within the server its hosted but not from outside. In order to access it from outside we need to update the context.xml file.

9.    ```bash
      find / -name context.xml
      ```
   
 <img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/224f10f1-b851-466f-a105-75b96d438dfc" alt="image" width="750" height="130" />

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

<img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/cee6eba0-5474-41bf-8815-1d475bfef516" alt="image" width="800" height="300" />

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