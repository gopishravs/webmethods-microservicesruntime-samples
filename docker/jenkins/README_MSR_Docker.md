## Work in progess ..

# Microservices Runtime  with Docker/Jenkins

This project allows user to quickly install Software AG Microservices Runtime, install update manager, install fixes, configure Mircoservices Runtime , build Mircoservices Runtime docker image and push the image into DockerHub using Jenkins pipeline.

## Getting Started
### Prerequisites

  Install below softwares 
 * Docker
 * Jenkins
 
**Note:** If above softwares were already installed, check 'Installing Docker' and 'Installing Jenkins and providing sudo sccess' sections below to verify all steps were covered and scroll down to 'one time setup'
### Installing Docker (CentOS 7 / RHEL 7)

A step by step series of examples that tell you how to install Docker in CentOS 7 / RHEL 7

Install Docker:
```
sudo yum install docker
```


 First remove older version of docker (if any):
```
sudo yum remove docker docker-common docker-selinux docker-engine-selinux docker-engine docker-ce
```

Install required packages
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

 Configure the docker-ce repo
```
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
Install docker-ce
```
sudo yum install docker-ce
```
Start Docker using systemctl
```
sudo systemctl start docker
```
or Start Docker using service
```
sudo service docker start
```
Login to Docker (Reference - https://docs.docker.com/engine/reference/commandline/login/)
```
sudo docker login -u [username] 
```

### Installing Jenkins and providing sudo sccess

A step by step series of examples that tell you how to install Jenkins and provide sudo access to run docker

Add the Jenkins repository to the yum repos, and install Jenkins from here.
```
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
sudo yum install jenkins
```

Jenkins requires Java in order to run, yet certain distros don't include this by default. To install the Open Java Development Kit (OpenJDK) run the following:
```
sudo yum install java
```

Start Jenkins
```
sudo service jenkins start
```

Launch Jenkins as below and follow instructions in browser to change password and set up account
```
http://localhost:8080
```
**Note:** Jenkins requires sudo permissions to run docker commands

Open the file sudoers
```
sudo vi /etc/sudoers 
```
Add below line to the file
```
jenkins ALL=(ALL) NOPASSWD: ALL
```


### One time setup
 * Create folder /opt/softwareag/resources
 * Clone current repository into /opt/softwareag/resources
 * cd into /opt/softwareag/resources and it would contain MSR_Docker folder, msrInstallerLinuxScript_10_3.txt, msrInstallerLinuxScript_10_5.txt, sum.txt, test.sh, wm-deploy.jar 
 * Download Software AG Installer(.bin) file for Linux from https://empower.softwareag.com/Products/DownloadProducts/sdc/default.aspx  into /opt/softwareag/resources
 * Download Software AG Update Manager Installer for Linux from https://empower.softwareag.com/Products/DownloadProducts/sdc/default.aspx  into /opt/softwareag/resources
 * Copy license file(licenseKey.xml) into /opt/softwareag/resources
 * Install SoftwareAGUpdateManagerInstaller20190930(LinuxX86).bin (version- 10.3) in any desired location (lets say /opt/softwareag/sagsum)
 * Execute UpdateManagerCMD.sh from  /opt/softwareag/sagsum/bin
 * Select option 9. Password Encryption, type Empower password to encrypt the password 
 * Copy the encrypted password(lets say "abcdefgh") and replace empowerPwd as below format in sum.txt (if you follow above steps you would find sum.txt under /opt/softwareag/resources )
      *  empowerPwd=abcdefgh
 * If jenkins is installed, copy 'MSR_Docker' folder into '/var/lib/jenkins/jobs' and restart the jenkins
   ```
   sudo service jenkins restart
   ```
  * After restart, click "MSR_Docker" in Jenkins dashboard (http://localhost:8080).
  * Click "Build with Parameters" from the options in the left
  * Provide valid parameters and click build.
   
## Jenkins Pipeline stages
##### Install MSR  
  Installs the MSR based on the parameters specified for Installer Configurations
##### Install SUM  
 Installs Update Manager based on specified bin file
##### Install Fixes
 Installs all the fixes to the installed MSR

##### Configure MSR
 This step is conditional. If APPLY_CONFIGURATION is set to YES then based on MSR Pre-Configurations, the configurations will be applied to the installed MSR

##### Create Docker File
 Creates Docker file for the installed MSR with latest fixes and optionally configurations.

##### Build Docker Image
 Builds docker image from docker file created from previous step.

##### Start MSR
 Starts the MSR in port 5555 within the  docker container 

##### Docker Info
Displays the docker images and process status of the running container

##### Test
Test the running container based on test.sh 

##### Stop MSR
Stop the running container

##### Push docker image
Push the docker image into specified Docker hub URL

##### Cleanup
Clean up the environment such as removing containers and images.


## Jenkins Parameters 
| Parameter               	| Description                                                                                                              	| Default                                                  	| Possible Values                                          	| Required                      	|
|-------------------------	|--------------------------------------------------------------------------------------------------------------------------	|----------------------------------------------------------	|----------------------------------------------------------	|-------------------------------	|
| SAG_DIR                 	| Directory to store resources such as SAG installer bin, SAG installer script, license key file, SUM script, test script. 	| /opt/softwareag/resources                                	|                                                          	| Yes                           	|
| INSTALL_FIXES                	| Installs fixes if set to YES                                                                              	| YES                                                       	|                                                          	| Yes                           	|
| APPLY_CONFIGURATION      	| Applies configuration if set to YES                                                                               	| NO                                                      	|                                                          	| Yes                           	|
| SAG_EMPOWER_USERNAME    	| Empower Username                                                                                                         	|                                                          	|                                                          	| Yes                           	|
| SAG_EMPOWER_PASSWORD    	| Empower Password                                                                                                         	|                                                          	|                                                          	| Yes                           	|
| SAG_INSTALLER_BIN       	| SAG installer bin file name                                                                                              	| SoftwareAGInstaller-Linux_x86_64.bin                     	|                                                          	| Yes                           	|
| SAG_INSTALLER_SCRIPT    	| SAG installer script file name                                                                                           	| msrInstallerLinuxScript_10_5.txt                         	| msrInstallerLinuxScript_10_5.txt                         	| Yes                           	|
|                         	|                                                                                                                          	|                                                          	| msrInstallerLinuxScript_10_3.txt                         	|                               	|
| SAG_INSTALLER_SANDBOX   	| Sandbox URL                                                                                                              	| https\://sdc.softwareag.com/cgi-bin/dataservewebM105.cgi 	| https\://sdc.softwareag.com/cgi-bin/dataservewebM105.cgi 	| Yes                           	|
|                         	|                                                                                                                          	|                                                          	| https\://sdc.softwareag.com/cgi-bin/dataservewebM103.cgi 	|                               	|
| SAG_MSR_DIR             	| MSR installation directory                                                                                               	| /opt/softwareag/msr                                      	|                                                          	| Yes                           	|
| SAG_MSR_LICENSE_FILE    	| license file name                                                                                                        	| licenseKey.xml                                           	|                                                          	| Yes                           	|
| SAG_MSR_DEFAULT_PORT    	| MSR default port                                                                                                         	| 5555                                                     	|                                                          	| Yes                           	|
| SAG_MSR_DIAGNOSTIC_PORT 	| MSR diagnostic port                                                                                                      	| 9999                                                     	|                                                          	| Yes                           	|
| SAG_SUM_INSTALLER_BIN   	| SAG update manager installer bin file name                                                                               	| SoftwareAGUpdateManagerInstaller20191101(LinuxX86).bin   	| SoftwareAGUpdateManagerInstaller20191101(LinuxX86).bin   	| Yes                           	|
|                         	|                                                                                                                          	|                                                          	| SoftwareAGUpdateManagerInstaller20190930(LinuxX86).bin   	|                               	|
| SAG_SUM_DIR             	| SUM installed directory                                                                                                  	| /opt/softwareag/sagsum                                   	|                                                          	| Yes                           	|
| SAG_SUM_SCRIPT          	| SUM update manager script file name                                                                                      	| sum.txt                                                  	|                                                          	| Yes                           	|
| ACDL_FILE               	| ACDL filename to configure MSR                                                                                           	| isconfiguration.acdl                                     	|                                                          	| Depends on APPLY_CONFIGURATION 	|
| ACDL_BIN_FILE           	| ACDL package (.zip) filename to configure MSR                                                                            	| isconfiguration.zip                                      	|                                                          	| Depends on APPLY_CONFIGURATION 	|
| SAG_TEST_SCRIPT         	| Test script file name                                                                                                    	| test.sh                                                  	|                                                          	| Yes                           	|
| DOCKER_REPO_URL         	| Docker Repository                                                                                                        	|                                                          	|                                                          	| Yes                           	|
| DOCKER_TAG              	| Docker Tag                                                                                                               	| 10.5.0.2                                                 	|                                                          	| Yes                           	|
| DOCKER_BASE_IMAGE_NAME  	| Docker image name                                                                                                        	| centos:7                                                 	|                                                          	| Yes                           	|                                                  
## Examples
### Jenkins Parameters for Microservices Runtime 10.5 

   
| Parmeters               	| Values                                                            	|
|-------------------------	|-------------------------------------------------------------------	|
| SAG_DIR                 	| /opt/softwareag/resources                                         	|
| INSTALL_FIXES                	| YES                                                                	|
| APPLY_CONFIGURATION      	| NO                                                               	|
| SAG_EMPOWER_USERNAME    	| empower username                                                  	|
| SAG_EMPOWER_PASSWORD    	| empower password                                                  	|
| SAG_INSTALLER_BIN       	| ${SAG_DIR}/SoftwareAGInstaller20191216-LinuxX86.bin               	|
| SAG_INSTALLER_SCRIPT    	| ${SAG_DIR}/msrInstallerLinuxScript_10_5.txt                       	|
| SAG_INSTALLER_SANDBOX   	| https\://sdc.softwareag.com/cgi-bin/dataservewebM105.cgi          	|
| SAG_MSR_DIR             	| /opt/softwareag/msr                                               	|
| SAG_MSR_LICENSE_FILE    	| ${SAG_DIR}/licenseKey.xml                                         	|
| SAG_MSR_DEFAULT_PORT    	| 5555                                                              	|
| SAG_MSR_DIAGNOSTIC_PORT 	| 9999                                                              	|
| SAG_SUM_INSTALLER_BIN   	| ${SAG_DIR}/SoftwareAGUpdateManagerInstaller20191101(LinuxX86).bin 	|
| SAG_SUM_DIR             	| /opt/softwareag/sagsum                                            	|
| SAG_SUM_SCRIPT          	| ${SAG_DIR}/sum.txt                                                	|
| ACDL_FILE               	| ${SAG_DIR}/isconfiguration.acdl                                   	|
| ACDL_BIN_FILE           	| ${SAG_DIR}/isconfiguration.zip                                    	|
| SAG_TEST_SCRIPT         	| ${SAG_DIR}/test.sh                                                	|
| DOCKER_REPO_URL         	| docker hub url                                                    	|
| DOCKER_TAG              	| 10.5.0.2                                                          	|
| DOCKER_BASE_IMAGE_NAME  	| centos:7                                                          	|

### Jenkins Parameters for Microservices Runtime 10.3

   
| Parmeters               	| Values                                                            	|
|-------------------------	|-------------------------------------------------------------------	|
| SAG_DIR                 	| /opt/softwareag/resources                                         	|
| INSTALL_FIXES                	| YES                                                               	|
| APPLY_CONFIGURATION      	| NO                                                               	|
| SAG_EMPOWER_USERNAME    	| empower username                                                  	|
| SAG_EMPOWER_PASSWORD    	| empower password                                                  	|
| SAG_INSTALLER_BIN       	| ${SAG_DIR}/SoftwareAGInstaller20191216-LinuxX86.bin               	|
| SAG_INSTALLER_SCRIPT    	| ${SAG_DIR}/msrInstallerLinuxScript_10_3.txt                       	|
| SAG_INSTALLER_SANDBOX   	| https\://sdc.softwareag.com/cgi-bin/dataservewebM105.cgi          	|
| SAG_MSR_DIR             	| /opt/softwareag/msr                                               	|
| SAG_MSR_LICENSE_FILE    	| ${SAG_DIR}/licenseKey.xml                                         	|
| SAG_MSR_DEFAULT_PORT    	| 5555                                                              	|
| SAG_MSR_DIAGNOSTIC_PORT 	| 9999                                                              	|
| SAG_SUM_INSTALLER_BIN   	| ${SAG_DIR}/SoftwareAGUpdateManagerInstaller20190930(LinuxX86).bin 	|
| SAG_SUM_DIR             	| /opt/softwareag/sagsum                                            	|
| SAG_SUM_SCRIPT          	| ${SAG_DIR}/sum.txt                                                	|
| ACDL_FILE               	| ${SAG_DIR}/isconfiguration.acdl                                   	|
| ACDL_BIN_FILE           	| ${SAG_DIR}/isconfiguration.zip                                    	|
| SAG_TEST_SCRIPT         	| ${SAG_DIR}/test.sh                                                	|
| DOCKER_REPO_URL         	| docker hub url                                                    	|
| DOCKER_TAG              	| 10.3.0.9                                                        	|
| DOCKER_BASE_IMAGE_NAME  	| centos:7                                                          	|

## Create Install Script for Microservices Runtime  
  To create install script( SAG_INSTALLER_SCRIPT  parameter) for Microservices Runtime from SAG Installer follow the steps in below links
 * 10.5 - https://documentation.softwareag.com/a_installer_and_update_manager/10-5_Software_AG_Installer_webhelp/index.html#page/using-sag-installer-webhelp/to-console_mode_18.html
* 10.3  -  https://documentation.softwareag.com/a_installer_and_update_manager/10-3_Software_AG_Installer_webhelp/index.html#page/using-sag-installer-webhelp/to-console_mode_17.html

## License

This project uses the Apache License Version 2.0. For details, see [the license file](LICENSE).

For more information about Microservices Runtime, see the official Software AG Microservices Runtime documentation.

______________________
These tools are provided as-is and without warranty or support. They do not constitute part of the Software AG product suite. Users are free to use, fork and modify them, subject to the license agreement. While Software AG welcomes contributions, we cannot guarantee to include every contribution in the master project.	

Contact us at [TECHcommunity](mailto:technologycommunity@softwareag.com?subject=Github/SoftwareAG) if you have any questions.
