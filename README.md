Jenkins + Nexus CI/CD Setup on Ubuntu

Overview

This project demonstrates the setup of a Jenkins CI/CD pipeline with Maven and Nexus on an Ubuntu server. The pipeline builds a Java application, uploads artifacts to Nexus, and deploys them to Nginx.

Steps
1. Create Ubuntu Server

- Instance type: t2.medium

- Disk: 30 GB

2. Install Jenkins

Follow the official Jenkins installation guide:

https://www.jenkins.io/doc/book/installing/linux/

3. Install Maven

       sudo apt install maven -y


Version: 3.8.7

4. Install Nexus (Docker)
   
		sudo apt install docker.io -y
		docker pull sonatype/nexus3
		docker volume create nexus-data
		docker run -d -p 8081:8081 --name nexus -v nexus-data:/nexus-data sonatype/nexus3

6. Fetch Nexus Admin Password

		docker ps
		docker exec -it nexus /bin/bash
		cat /nexus-data/admin.password


Example output:

    da2a59b0-3c0d-4b8e-b309-fc5d3e0e7c0c

6. Install Jenkins Plugins

- Pipeline Maven Integration

- Nexus Artifact Uploader

7. Configure Maven in Jenkins

- Navigate to Manage Jenkins → Tools → Configure Maven

- Add new Maven installation:

		Name: M3
		MAVEN_HOME: /usr/share/maven

8. Configure Maven Settings in Jenkins


 1. Navigate: Managed Files → Add new config

 2. ID: Global-Setting

Add content:

	<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
	          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
	                              https://maven.apache.org/xsd/settings-1.0.0.xsd">
	  <servers>
	    <server>
	      <id>maven-releases</id>
	      <username>admin</username>
	      <password>admin123</password>
	    </server>
	    <server>
	      <id>maven-snapshots</id>
	      <username>admin</username>
	      <password>admin123</password>
	    </server>
	  </servers>
	</settings>

9. Create .m2 Folder on Ubuntu


		sudo mkdir -p /var/lib/jenkins/.m2
		sudo vim /var/lib/jenkins/.m2/settings.xml


Paste the Maven settings.xml content from step 8.

10. Edit pom.xml to Include Server IP

Example: 52.66.206.151

11. Jenkins Pipeline (Jenkinsfile)


		pipeline {
		    agent any
		
		    environment {
		        APP_NAME = "myapp"
		    }
		
		    stages {
		        stage('Checkout Code') {
		            steps {
		                git branch: 'master', url: 'https://github.com/ygminds73/maven-simple-master.git'
		            }
		        }
		
		        stage('Build with Maven') {
		            steps {
		                sh 'mvn clean package'
		            }
		        }
		
		        stage('Upload to Nexus') {
		            steps {
		                withMaven(maven: 'M3') {
		                    sh "mvn clean deploy -s /var/lib/jenkins/.m2/settings.xml -DskipTests"
		                }
		            }
		        }
		
		        stage('Download from Nexus') {
		            steps {
		                sh """
		                curl -u admin:admin@123 -O \
		                http://65.2.9.176:8081/repository/maven-releases/com/github/jitpack/maven-simple/0.2-SNAPSHOT/maven-simple-0.2-SNAPSHOT.jar
		                """
		            }
		        }
		
		        stage('Deploy to Nginx') {
		            steps {
		                sh """
		                sudo rm -rf /var/www/html/*
		                sudo cp target/maven-simple-0.2-SNAPSHOT.jar /var/www/html/
		                sudo systemctl restart nginx
		                """
		            }
		        }
		    }
		}

12. Install Nginx

		sudo apt install nginx -y

13. Pipeline First Build

- Set branch name: main

14. Pipeline Second Build

- Give permission to .m2 folder:

		sudo mkdir -p /var/lib/jenkins/.m2/repository
		sudo chown -R jenkins:jenkins /var/lib/jenkins/.m2


- Refresh Jenkins

- Restart Nexus container if needed

15. Verify Maven Password

- Check Jenkins Maven config

- Check /var/lib/jenkins/.m2/settings.xml on Ubuntu

16. Grant Jenkins Sudo Permissions

Edit sudoers file:

    sudo visudo


Add at the end:

    jenkins ALL=(ALL) NOPASSWD: ALL


Or limit commands:

	jenkins ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx, /bin/rm, /bin/cp
	sudo chown -R jenkins:jenkins /var/www/html

Notes

- Make sure Docker has enough memory for Nexus.

- Ensure Jenkins user owns .m2 and web directories.

- Always test builds incrementally.
