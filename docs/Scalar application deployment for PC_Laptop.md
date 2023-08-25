## 1. Prerequisites
Laptop/PC memory to be 8GB minimum.
Internet connectivity till the setup is done.
HDD/SDD 10GB free space.
Virtualization to be enabled.
wsl2 should be present on the local machine.
PAT token creation.
Docker Desktop installed.
Access to Scalar’s Container Registry (GitHub container registry access).

## 2. Virtualization
Check if virtualization is enabled on your PC/Laptop. Simply press Ctrl + Shift + Esc keys to open the Task Manager. 
Click on the Performance tab and under CPU, you will find information about Virtualization on your desktop/laptop. 
If it says Enabled, then Virtualization is turned on. Else follow the steps mentioned in 
https://www.minitool.com/news/enable-virtualization-windows-10.html.

## 3. WSL installation: (Windows Subsystem for Linux)
Command to install wsl:
```bash
wsl    --install    -d    Ubuntu-20.04
```
Restart your machine to complete installation. After restarting PC/Laptop, check the wsl version.
```bash
wsl  -l   -v
```
If under ‘Version’ it is displayed 1 for Ubuntu-20.04, then update to Version 2, using the following command
```bash
wsl    --set-version   Ubuntu-20.04    2
```
Once updated, update the wsl linux installation to the latest update by executing the following command assuming wsl is not running.
```bash
wsl
sudo   apt-get   update
```

## 4. Docker installation
Install Docker desktop. This installation allows executing of Front end and backend applications in containerized form. Pls refer to the following URL.
https://docs.docker.com/desktop/install/windows-install/

## 5. PAT token
Please create a GitHub PAT token from your GitHub account. Refer to the following link.   
Need to generate a Classic token with at least read access.
https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens

## 6. Client container registry login
Open wsl on command prompt, if not already started. 
This is required only once to fetch the Scalar docker images from Scalar’s GitHub container registry. 

Please contact admin to check if you have requisite permission to access the Scalar’s GitHub container registry and refer above to generate your PAT token.

Open wsl on command prompt as:
```bash
wsl
```
Logout of GitHub, if logged in earlier by entering the following command.
```bash
docker logout ghcr.io
```
Login to Scalar’s Container Registry (Github) by executing following commands. Replace your PAT token in the following command in 
{your personal access token}
```bash
    export CR_PAT={your personal access token}
    echo  $CR_PAT |  sudo docker  login ghcr.io  -u {userName}  --password-stdin
```

## 7. Project download

Download the Scalar Demo Project from GitHub. 

You can either clone the project from Github or download the zipped project from GitHub and unzip it. 
https://github.com/PerceptCS/Scalar_Filemanager_Demo

Move the downloaded project folder to another location of your choice.
Once downloaded, run wsl. 
From wsl  go to the Scalar Demo Project folder from root path and run the docker compose file as mentioned in the next sections. 
This is one time activity only.

## 8. Starting containerised applications:

Before Running compose file(i.e starting applications), we need to change two settings as below:

These settings are for configuring the application for local storage or S3 storage.

<ins>Changing the settings</ins>:
Settings for local (default configured) or for cloud is changed in file docker-compose-ledger-auditor.yml which is located in the following folder path
```bash
....\Scalar_Filemanager_Demo-main\scalardl-samples-mysql-ledger-auditor
```
In the above statement, … denotes the folder path where the downloaded project is copied to. Changing the settings is one time activity. 
Please refer to the partial text  image of docker compose file for reference. The area of text which is to be changed is  highlighted in blue color.

![compose_modification](/docs/assets/images/local_config/docker_compose_part.jpg)

- A) Configuring the project for files stored in cloud:
Open the docker-compose-ledger-auditor.yml  file in a simple text editor such as Notepad. Go to the following location as mentioned below
```bash
    environment:
      - IS_ON_LOCAL_MACHINE=true
      - STORAGE_PROVIDER_TYPE=LOCAL 
```
When the project is downloaded, above are the default settings, i.e file storage in local. Now change it as follows. Please do not add any unnecessary spaces before and after the = sign.
bash```
	environment:
      - IS_ON_LOCAL_MACHINE=false
      - STORAGE_PROVIDER_TYPE=AWS_S3 
```
- B) Configuring the project for files stored in local:
This will be the default configuration when the project is downloaded, however if you have configured it for file storage on cloud and now would want 
to change to file storage on local configure it as follows:
```bash
    environment:
      - IS_ON_LOCAL_MACHINE=true
      - STORAGE_PROVIDER_TYPE=LOCAL 
```

Current the projects compose file uses default environment variables for  scalar_filemanager application that are shown below,
     1) IS_ON_LOCAL_MACHINE=true 
     2) STORAGE_PROVIDER_TYPE=LOCAL

Environment Variable: **IS_ON_LOCAL_MACHINE**

This environment variable serves as an indicator of whether the application is running on a local system or not. Its purpose is to influence the behavior of the application based on its value.

When **IS_ON_LOCAL_MACHINE** is set to true:
Signup Without OTP Validation: The application will allow user sign-up without requiring OTP (One-Time Password) validation.
Internet Connectivity: It implies that the application might not have internet connectivity. Therefore, the process of sending email for OTP is bypassed.
Fault Injection Limitation: The option to inject faults using changeFileOnS3 and fileURL in Scalar DB is unavailable in this mode.

When **IS_ON_LOCAL_MACHINE** is set to false, the application operates under default conditions, which may include:

Regular Signup with OTP Validation: User sign-up will follow the standard procedure, including OTP (One-Time Password) validation.
Internet Connectivity: The application assumes internet connectivity and proceeds with sending OTP emails if required.
Fault Injection Option: The option to inject faults using changeFileOnS3 and fileURL in Scalar DB may be available

Environment Variable: **STORAGE_PROVIDER_TYPE**

This environment variable specifies the type of storage provider that the application will use to save files. It offers two options:

**LOCAL**: When set to LOCAL, the application will utilize local storage for file storage purposes.

**AWS_S3**: Alternatively, if set to AWS_S3, the application will employ Amazon Web Services' S3 (Simple Storage Service) for file storage.

Configure above environment variable wisely,

Run/execute the compose file as follows:
docker  compose   -f   docker-compose-ledger-auditor.yml   up    -d
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

## 9. Starting the application for the first time
Once you have confirmed that your setup is complete properly as above, please start your application for the first time as below:
Open windows command prompt as Administrator only. Go to the following location
```bash
      C:\Program Files\Google\Chrome\Application
```

Run following command
```bash
chrome.exe --disable-web-security --disable-gpu --user-data-dir=~/chromeTemp
```
From the newly opened Chrome browser session, enter the following link
```bash
     http://localhost/fmui-local/ 
```

Let the web page be displayed. Once the web page is displayed, then your first time setup process is completed. Shown below how the web page 
will be displayed.

![chromepage](/docs/assets/images/local_config/web_display.jpg)

## 10. Executing the application from Chrome browser:
Open windows command prompt as Administrator only. Go to the following location.
```bash
      C:\Program Files\Google\Chrome\Application
```
Run following command
```bash
          chrome.exe --disable-web-security --disable-gpu --user-data-dir=~/chromeTemp
```
From the newly opened Chrome browser session, enter the following link
```bash
    http://localhost/fmui-local/ 
```
If executing for the first time, then register and then login. Start using the application.

## 11. Stopping the application:
Go to the Scalar Demo Project folder from the Windows command prompt. Run the following command. This will shut down all the docker containers and free its memory. It is not necessary to start wsl.
In case your application was started for LOCAL/AWS-S3 file storage, then run the following command for shutdown. 
docker  compose  -f  docker-compose-ledger-auditor.yml  down  
Alternatively, applications can be shutdown from wsl by going to the project folder’s location and using the above command. However, just prefix the command with sudo.

## 12. Server deployment:
As of now the front end and backend application is deployed on our AWS EC2 server. Also the files uploaded to the cloud via application deployed on our server are saved in our AWS S3 bucket.
	In case you need to deploy the application to your hosted server, then we will have to give you the application configured to your AWS S3 bucket. Hence you need to have a server on cloud and a S3 bucket to store files remotely.












