# Scalar application deployment on cloud.

## 1. Prerequisites
   1. Server must be Ubuntu 20.04 minimum with minimum 4GB RAM. 
   2. Access to server on cloud. 
   3. Internet connectivity.    
   4. HDD/SDD 10GB free space.     
   5. PAT token creation.    
   6. Docker installed on server.    
   7. Access to Scalar’s Container Registry (GitHub container registry access).    

## 2. Docker installation
Install Docker for Ubuntu. This installation allows executing of Frontend and backend applications in containerized form. Pls refer to the following URL.
https://docs.docker.com/engine/install/ubuntu/

## 3. PAT token
Please create a GitHub PAT token from your GitHub account. Refer to the following link.   
Need to generate a Classic token with at least read access.
https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens

## 4. Client container registry login
Please contact admin to check if you have requisite permission to access the Scalar’s GitHub container registry and refer above to generate your PAT token.

Open linux terminal, logout of GitHub, if logged in earlier by entering the following command.
```bash
docker logout ghcr.io
```
Login to Scalar’s Container Registry (Github) by executing following commands. Replace your PAT token in the following command in 
{your personal access token}
```bash
    export CR_PAT={your personal access token}
    echo  $CR_PAT |  sudo docker  login ghcr.io  -u {userName}  --password-stdin
```

## 5. Project download
Download the Scalar Demo Project from GitHub. 

You can either clone the project from Github or download the zipped project from GitHub and unzip it. 
https://github.com/PerceptCS/Scalar_Filemanager_Demo

Move the downloaded project folder to another location of your choice.     

From terminal go to the Scalar Demo Project folder from the root path. 

## 6. Configuring docker compose file to setup Applications:

Before Running compose file(i.e starting applications), we need to change two settings as below:

These settings are for configuring the application for local storage (configured as default in the file)  or for cloud storage (S3 storage).

<ins>Changing the settings</ins>:
Settings for local (default configured) or for cloud is changed in file docker-compose-ledger-auditor.yml which is located in the following folder path
```bash
....\Scalar_Filemanager_Demo-main\scalardl-samples-mysql-ledger-auditor
```
In the above statement, … denotes the folder path where the downloaded project is copied to.     
Refer part of this file as below. The settings to be changed are highlighted.

![compose_modification](/docs/assets/images/local_config/docker_compose_part.jpg)

   Configuring the project for files stored in cloud:    
     Open the docker-compose-ledger-auditor.yml  file in a simple text editor such as Notepad. Go to the following location as mentioned below
```bash
    environment:
      - IS_ON_LOCAL_MACHINE=true
      - STORAGE_PROVIDER_TYPE=LOCAL 
```
     When the project is downloaded, above are the default settings, i.e file storage in local. Now change it as follows. Please do not add 
     any unnecessary spaces before and after the = sign.

```bash
    environment:
      - IS_ON_LOCAL_MACHINE=false
      - STORAGE_PROVIDER_TYPE=AWS_S3 
```
**Note**:    
       1. The default settings are for file storage on local device. This configuration is for running the applications on the desktop or laptop when there are internet connectivity restrictions.    
       2. If you want to use cloud storage, the settings need to be changed as mentioned above.     
	  
   Configuring the UI docker image:
   Referring to the image below, replace the default standard image with the image created specifically for the cloud IP address.
![compose_modification2](/docs/assets/images/local_config/docker_compose_part2.jpg)	   

## 7 Significance of the Environment Variables:
The compose file of this project uses two environment variables for setting storage configuration.
     1) IS_ON_LOCAL_MACHINE=true 
     2) STORAGE_PROVIDER_TYPE=LOCAL

A) Environment Variable: **IS_ON_LOCAL_MACHINE**     

This environment variable serves as an indicator of whether the application is running on a local system or not. Its purpose is to influence the 
behavior of the application based on its value.

When **IS_ON_LOCAL_MACHINE** is set to true:      
Signup Without OTP Validation: The application will allow user sign-up without requiring OTP (One-Time Password) validation.   
Internet Connectivity: It implies that the application might not have internet connectivity. Therefore, the process of sending email for OTP is bypassed.

Fault Injection Limitation: While performing Inject Fault operation, the following options to inject fault are not available:     
      1. File On S3    
      2. File URL in scalar DB  


When **IS_ON_LOCAL_MACHINE** is set to false, the application assumes internet connectivity , hence following points are assumed:

Regular Signup with OTP Validation: User sign-up will follow the standard procedure, including OTP (One-Time Password) validation.

Internet Connectivity: The application assumes internet connectivity and proceeds with sending OTP emails if required.

Fault Injection Option: All the options displayed on the Application UI are available.

B) Environment Variable: **STORAGE_PROVIDER_TYPE**

This environment variable specifies the type of storage provider that the application will use to save files. It offers two options:     

     **LOCAL**: When set to LOCAL, the application will utilize local storage for file storage purposes.  

     **AWS_S3**: Alternatively, if you set to AWS_S3, the application will employ Amazon Web Services' S3 (Simple Storage Service) for file storage.

Configure above environment variable wisely,

Then, Run/execute the compose file using command as :
```bash
docker  compose   -f   docker-compose-ledger-auditor.yml   up    -d
```
The following actions take place when the compose file is executed.
- Ledger and auditor certificates have been registered with each other. So the ledger and auditor can talk.
- Respective Schema has been loaded (as configured in compose file) in Ledger Container, Auditor Container
- Multiple ports gets exposed like 9901,50051,50052(ledger-envoy) ,9902,40051,40052(scalar-envoy) and much more on that instance 
- Once all Dockers are in either Created/Started/Healthy state, then can run the File Manager application from the Chrome browser.

Following is the snapshot of  docker compose file execution after a few minutes.

![dockercompose](/docs/assets/images/local_config/compose_file_up1.jpg)

Once the command prompt returns after running the project’s docker compose file, run the following command after a few minutes.
```bash
docker    ps
```
![dockercompose](/docs/assets/images/local_config/compose_file_up2.jpg)

As seen from the above image there will be a total of 12 docker containers running. Ensure that 5 containers' status must be displayed 
as (healthy) as shown in the STATUS column.

## 8 Before running the application
   1. Assuming that the deployment is on AWS EC2 instance modify the security group to allow the following ports, 
      80, 8092 and 8093.
	  
   2. At server, add an entry to the hosts file located in folder /etc as follows and save the file. 
      ```bash   
      172.17.0.1   host.docker.internal  
	  ```
	  
   3. Restart the server by running the following command.
      ```bash
      sudo reboot +0
	  ```
	  
   4. After few minutes when the server starts, login to the server and go to the location where the docker compose file is located.

## 9 Using your File Manager application
Whenever you want to run the application, please perform following steps:

Go to the projects folder as mentioned above and run the compose file in the following two steps.
```bash
docker  compose   -f   docker-compose-ledger-auditor.yml   down
```
Wait for a few minutes till all dockers are shut down and then run the following command.
```bash
docker  compose   -f   docker-compose-ledger-auditor.yml   up    -d
```

From Chrome browser session, enter the following link
```bash
    http://ip-address/fmui-local/ 
```
In above url, the 'ip-address' refers to the server ip address.

Following image shows how login page looks.
![chromepage](/docs/assets/images/local_config/web_display.jpg)