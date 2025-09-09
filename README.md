Steps -

1. Create Ubuntu server -> T2 medium, 30 GB

2. Install Jenkins on ubuntu server. -

   https://www.jenkins.io/doc/book/installing/linux/

3. Install maven 3.8.7

   # apt install maven -y

4. Nexus Installation on docker image
   
  # apt install docker.io -y

  # docker pull sonatype/nexus3

  # docker volume create nexus-data

  # docker run -d -p 8081:8081 --name nexus \
  -v nexus-data:/nexus-data \
  sonatype/nexus3

5. Fetch password -

# docker ps

# docker exec -it nexus /bin/bash

# cat /nexus-data/admin.password

da2a59b0-3c0d-4b8e-b309-fc5d3e0e7c0c


6. Install below Plugins on Jenkins server

   1. Pipeline Maven Integration

   2. Nexus Artifact Uploader


7. Navigate to Manage Jenkins -> Tools -> Configure Maven installation ->

   Name : M3
   MAVEN_HOME : /usr/share/maven

8.  setting of  Jenkins ->
     Managed Files -> Add new config
     ID Name :(Global-Setting)
     -> content -> 111 <add on that lines below code >
    

    mkdir -p /var/lib/jenkins/.m2
    sudo nano  /var/lib/jenkins/.m2/settings.xml
    <settings xmlns="http://maven.apache.org/SETTTUGs/1.0.0"
    		xmlns : xsi="http://www.w3.org/2001/XMLSchema-instance"
			xsi : schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
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
  

9. Login to Jenkins servr


# ls -l /var/lib/jenkins/.M2/
# mkdir -p /var/lib/jenkins/.m2
# vim /var/lib/jenkins/.m2/settings.xml

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


10. Edit POM.xml for server IP

 ex. 52.66.206.151


11. Write your Jenkinsfile

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



after adding the pipeline :

then create the .m2 folder and inside of that folder create the settings.xml file .

settings.xml:
 mkdir -p /var/lib/jenkins/.m2
    sudo nano  /var/lib/jenkins/.m2/settings.xml
    <settings xmlns="http://maven.apache.org/SETTTUGs/1.0.0"
    		xmlns : xsi="http://www.w3.org/2001/XMLSchema-instance"
			xsi : schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
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


after that install the nginx on machine :

# apt install nginx -y 



first build :
give branch name as main in pipeline 

second build : 
give the permission to the file in .m2 (Jenkins folder) :
sudo mkdir -p /var/lib/jenkins/.m2/repository
sudo chown -R jenkins:jenkins /var/lib/jenkins/.m2

refresh the jenkins and close the nexux and start again .


next build :

check the password in maven config in jenkins and also in jenkins server on ubuntu .

last : 

-open file 
#sudo visudo
add line  to the last of file 
#jenkins ALL=(ALL) NOPASSWD: ALL


build again :

12. # vim /etc/sudoers




jenkins ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx, /bin/rm, /bin/cp
or
sudo chown -R jenkins:jenkins /var/www/html
