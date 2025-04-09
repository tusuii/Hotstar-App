--------------------------------------------------------------------------------------------------------------------
Jenkins + GitHub + Maven + Nexus + SonarQube + Tomcat + S3 - Pipeline Project
--------------------------------------------------------------------------------------------------------------------
HOTSTAR APP
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Jenkins Setup - Amazon Linux 2, t2.micro
	Integrate with Git & GitHub
	Integrate with Maven
Tomcat Setup - Amazon Linux 2, t2.micro
SonarQube Setup - Amazon Linux 2, t2.medium
Nexus Setup - Ubuntu 24.04, t4.medium
	Artifacts in S3 Bucket (included)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------
----------------
Step 1:
----------------
Launch 2 Amazon Linux 2 AMI, t2.micro, 10 GB (Instance 1 Name: Jenkins Server, Instance 2 Name: Tomcat Server)
Launch 1 Amazon Linux 2 AMI, t2.medium, 10 GB (Instance Name: Sonar Server)
Launch 1 Ubuntu 24.04 AMI, t2.medium, 10 GB (Instance Name: Nexus Server)

----------------
Step 2:
----------------
Connect to Jenkins Server and install Jenkins using below commands; sudo -s ----> sudo yum update -y
Setup the hostname for each server ----> hostnamectl set-hostname JenkinsServer ----> sudo -i
 
vi jenkins.sh ----> Paste the below commands in the file ---->

#STEP-1: Installing Git and Maven
yum install git maven -y

#STEP-2: Repo Information (jenkins.io --> download -- > redhat)
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

#STEP-3: Download Java 17 and Jenkins
sudo yum install java-17-amazon-corretto -y
yum install jenkins -y

#STEP-4: Start and check the JENKINS Status
systemctl start jenkins.service
systemctl status jenkins.service

#STEP-5: Auto-Start Jenkins
chkconfig jenkins on

----> esc ----> :wq ----> sh jenkins.sh ----> You will see jenkins is running ----> Access jenkins and setup jenkins

----------------
Step 3:
----------------
Connect to Sonar Server and install SonarQube using below commands; sudo -s ----> sudo yum update -y
Setup the hostname for each server ----> hostnamectl set-hostname SonarServer ----> sudo -i

vi sonar.sh ----> Paste the below commands in the file ---->
#! /bin/bash
cd /opt/
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.9.6.50800.zip
unzip sonarqube-8.9.6.50800.zip
amazon-linux-extras install java-openjdk11 -y
useradd sonar
chown sonar:sonar sonarqube-8.9.6.50800 -R
chmod 777 sonarqube-8.9.6.50800 -R
su - sonar


----> esc ----> :wq ----> sh sonar.sh ----> You will see sonar is running ----> Access sonar and setup sonar

# use the below command manually after installation
sh /opt/sonarqube-8.9.6.50800/bin/linux-x86-64/sonar.sh start ----> You will see "Sonar Started"

Open Port no. 9000 for the server and setup SonarQube 
Username: admin
Password: admin


----------------
Step 4:
----------------
Connect to Tomcat Server and install SonarQube using below commands; sudo -s ----> sudo yum update -y
Setup the hostname for each server ----> hostnamectl set-hostname TomcatServer ----> sudo -i

vi tomcat.sh ----> Paste the below commands in the file ---->
amazon-linux-extras install java-openjdk11 -y

----> esc ----> :wq ----> sh tomcat.sh ----> You will see tomcat is running ----> Access tomcat

Open Port no. 8080 for the server and setup tomcat

----------------
Step 5:
----------------
Connect to Nexus Server and install Nexus using below commands; sudo -s ----> sudo apt-get update -y
Setup the hostname for each server ----> hostnamectl set-hostname NexusServer ----> sudo -i

Execute the below commands one by one ---->
## #1: Install OpenJDK 17 on Ubuntu 24.04 LTS
apt install openjdk-17-jdk openjdk-17-jre -y

##Download the SonaType Nexus on Ubuntu using wget
cd /opt 
sudo wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz   


##Extract the Nexus repository setup in /opt directory
tar -zxvf latest-unix.tar.gz     

##Rename the extracted Nexus setup folder to nexus
sudo mv /opt/nexus-3.77.0-08/ /opt/nexus

##As security practice, not to run nexus service using root user, so lets create new user named nexus to run nexus service
sudo adduser nexus  ----> give password i.e root1234
Press ENTER for 5 times
Press Y
You will see 'Adding user 'nexus' to group 'users'

##To set no password for nexus user open the visudo file in ubuntu
sudo visudo

##Add below line after  "Defaults" 		use_pty
nexus ALL=(ALL) NOPASSWD: ALL
Save and exit (Control + X, Press Y, Click Enter)

##Give permission to nexus files and nexus directory to nexus user
sudo chown -R nexus:nexus /opt/nexus
sudo chown -R nexus:nexus /opt/sonatype-work        

##To run nexus as service at boot time, open /opt/nexus/bin/nexus.rc file, uncomment it and add nexus user as shown below
sudo nano /opt/nexus/bin/nexus.rc
Uncomment the line which you see after opening the file
In the double quotes, keep "nexus". The finale line should be as shown below
run_as_user="nexus"   
Save and exit

##To Increase the nexus JVM heap size, you can modify the size as shown below
vi /opt/nexus/bin/nexus.vmoptions

Remove the first 15 lines i.e upto the space just before the "#additional vmoptions needed for java9+". Remember, dont delete "#additional vmoptions needed for java9+" line
Paste the below content above "#additional vmoptions needed for java9+" line

-Xms1024m
-Xmx1024m
-XX:MaxDirectMemorySize=1024m

-XX:LogFile=./sonatype-work/nexus3/log/jvm.log
-XX:-OmitStackTraceInFastThrow
-Djava.net.preferIPv4Stack=true
-Dkaraf.home=.
-Dkaraf.base=.
-Dkaraf.etc=etc/karaf
-Djava.util.logging.config.file=/etc/karaf/java.util.logging.properties
-Dkaraf.data=./sonatype-work/nexus3
-Dkaraf.log=./sonatype-work/nexus3/log
-Djava.io.tmpdir=./sonatype-work/nexus3/tmp 

Save and exit

##To run nexus as service using Systemd
sudo vi /etc/systemd/system/nexus.service ----> Paste the below content ---->

[Unit]
Description=nexus service
After=network.target

[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
User=nexus
Restart=on-abort

[Install]
WantedBy=multi-user.target 

Save and exit


To start the nexus Service
sudo systemctl start nexus 
sudo systemctl enable nexus 
sudo systemctl status nexus  

Open port no. 8081 and access nexus

----------------
Step 6:
----------------
6.1. Install below plugins in Jenkins
Pipeline stageview, Deploy to container, S3 publisher, nexus artifact uploader, SonarQube Scanner, Sonar Quality Gate, Maven Integration

'Check' Restart Jenkins after the above plugins got installed

6.2. Setup Tomcat Creds in Jenkins
Jenkins ----> Manage Jenkins ----> Security ----> Credentials ----> Click on 'global' ----> Add credentials ----> Kind: Username with Password, Scope: Global, username: admin, Password: admin, ID: tomcat-creds, Description: tomcat-creds ----> Create 

6.3. Create pipeline job
Repo URL:  (HOTSTAR App)

For the stage 'Deploy to Tomcat' generate the pipeline syntax;
Open Pipeline Syntax in new tab ----> Sample step: deploy: Deploy war/ear to a container, WAR/EAR Files: **/*.war, Context Path: hotstarapp, Add container: Select 'Tomcat 9.x Remote', Credentials: Select the tomcat creds from dropdown, Tomcat URL: Paste upto 8080/ ----> Generate pipeline script ----> Copy and paste in 'Deploy to Tomcat' stage

Paste the below script;
pipeline {
    agent any
    stages {
        stage('Git Checkout') {
            steps {
                git '<Repo URL>'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        } 
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Arifacts') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Deploy to Tomcat') {
            steps {
                <Paste The Script>
            }
        }
    }
}

----> Apply and Build the job ----> Job will be successful

Access the app in Tomcat browser. You will see hotstar app.

6.4. Lets push the artifact to S3 Bucket (To do this we need 'S3 Publisher' plugin which we have already installed)
To do this, we need to generate access and secret keys and we also need to create s3 bucket

IAM ----> Create IAM User with Administrator Access ----> Open user ----> 'Security Credentials' tab ----> Create access key ----> Use case: AWS CLI,  ----> Next ----> Create access key

S3 ----> Create a bucket with default settings but enable versioning

Lets configure access keys in jenkins ----> Manage jenkins ----> System Configuration ----> System ----> Scroll down to "Amazon S3 Profiles" ----> Add ----> Profile name: s3creds, Access key: <PasteAccessKey>, Secret key: <PasteSecretKey> ----> Apply ---->  Save

Open the job created ----> Open pipeline syntax (here we will generate script to push artifacts to S3) ----> Sample step: S3upload: Publish artifacts to S3 bucket, S3 profile: s3creds, Files to upload: **/*.war, Destination Bucket: <GiveTheBucketName>/ , Storage class: Standard, Bucket region: <SelectTheRegionofBucket>, 'check' Server side encryption ----> Generate pipeline script ----> Copy and paste in 'Artifacts to S3' stage

Paste the below script;
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git '<Repo URL>'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        } 
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Arifacts') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Deploy to Tomcat') {
            steps {
                <Paste The Script>
            }
        }
        stage('Artifact to S3') {
            steps {
                <Paste The Script>
            }
        }
    }
}

----> Apply and Build the job ----> Job will be successful ---> Goto S3 and check for artifact

6.5. Working with SonarQube
Goto SonarQube ----> Projects ----> Add project ----> Manually ----> Project key: hotstar, Display name: hotstar ----> Setup ----> Generate a token: hotstar ----> Generate ----> Copy the token ----> Lets configure the SonarQube credentials in Jenkins

97eea7608116feb3a4b6c61ed0c49b2b2cac80ab

Jenkins ----> Manage Jenkins ----> Security ----> Credentials ----> Click on 'global' ----> Add creds ----> Kind: Secret text, Scope: Global, Secret: Paste the secret copied from SonarQube, ID: sonar, Description: sonar ----> Create

Jenkins ----> Manage Jenkins ----> System Configuration ----> System ----> Scroll down to 'SonarQube servers' ----> 'Check' environment variables, Add SonarQube ----> Name: SonarQube, Server URL: <SonarQube URL> [only upto 9000] ----> Server authentication token: Select 'sonar' from dropdown ----> Apply ----> Save

Jenkins ----> Manage Jenkins ----> System Configuration ----> Tools ----> Scroll down to 'sonarqube Scanner Installations' ----> Add SonarQube scanner ----> Name: sonarscanner, 'Check' Install automatically, Version: 7.1.0.4889 ----> Apply ----> Save

Lets integrate SonarQube in the pipeline ----> Open the job ----> Paste the below script (In the script, i have added sonarqube-scanning' stage before the artifacts stage)

pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git '<Repo URL>'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        } 
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('SonarQube Scanning') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
                }
            }
        }
        stage('Artifacts') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Deploy to Tomcat') {
            steps {
                // <Paste The Script>
            }
        }
        stage('Artifact to S3') {
            steps {
                // <Paste The Script>
            }
        }
    }
}


----> Apply and Build the job ----> Job will be successful ---> Goto SonarQube dashboard and check it

6.6. Nexus Configuration
Goto Nexus tab in browser ----> Signin ----> Username: admin, To get the password, copy the path seen in the nexus browser ----> Goto the nexus connected tab in Moba ----> cat <Paste The Path> ----> Copy the password. dont copy "root@nexus" ----> Paste the password in nexus browser ----> Signin ----> Set the new password ----> Enable anonymous access ----> Signin

Click on settings icon ----> Click on 'repositories' ----> Create repo ----> Scroll down and Click on 'maven2 hosted' ----> Name: hotstar, Version policy: Snapshot, Layout policy: Strict, Content disposition: Inline, Blob store: default, Deployment policy: Allow redeploy ----> Create repository ----> You will see 'hotstar' repo ----> Lets configure the Nexus credentials in Jenkins

Jenkins ----> Manage Jenkins ----> Security ----> Credentials ----> Click on 'global' ----> Add creds ----> Kind: Username with Password, Scope: Global, Username: admin, Password: <Enter The Password of SonarQube>,ID: nexus, Description: nexus ----> Create

Lets integrate Nexus in the pipeline ----> Open the job ----> Open pipeline syntax in new tab ----> Sample step: nexusArtifactUploader: Nexus Artifact Uploader, Nexus Version: Nexus3, Protocol: HTTP, Nexus URL: <Paste The Nexus URL> [Ex: IP:8081], Credentials: Select 'nexus' from dropdown, GroupId: in.kastro (this is in pom.xml file), Version: 8.3.3-Snapshot (this is in pom.xml file), Repository: hotstar (This name should be same as created in the Nexus console), Artifacts: Add: Artifact ID: myapp (this is in pom.xml file), Type: .war, Classifier: <Leave Blank>, File: target/myapp.war (Note: here instead of path, we can also mention **/*.war) ----> Generate script ----> Copy the script and paste in the below pipeline stage of 'Nexus'
Paste the below script (In the script, i have added 'Nexus' stage after the Tomcat stage)

pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git '<Repo URL>'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        } 
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('SonarQube Scanning') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
                }
            }
        }
        stage('Artifacts') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Deploy to Tomcat') {
            steps {
                // <Paste The Script>
            }
        }
        stage('Nexus') {
            steps {
                // <Paste The Script>
            }
        }
        stage('Artifact to S3') {
            steps {
                // <Paste The Script>
            }
        }
    }
}

----> Apply and Build the job ----> Job will be successful ---> Goto Nexus dashboard and click on 'hotstar' ----> Copy the URL you see and paste in new tab ----> 	Click on 'browse' ----> You will see the war file got stored as artifact
Goto Tomcat and check for application


final jenkins script

pipeline {
    agent any
    stages {
        stage('Git Checkout') {
            steps {
                git 'https://github.com/tusuii/Hotstar-App.git'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        } 
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('SonarQube Scanning') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
                }
            }
        }
        stage('Arifacts') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Deploy to Tomcat') {
            steps {
               deploy adapters: [tomcat9(credentialsId: 'tomcat-creds', path: '', url: 'http://13.232.174.196:8080/')], contextPath: 'hotstarapp', war: '**/*.war'
            }
        }
        stage('Nexus') {
            steps {
               nexusArtifactUploader artifacts: [[artifactId: 'myapp', classifier: '', file: 'target/myapp.war', type: '.war']], credentialsId: 'nexus', groupId: 'in.subodh', nexusUrl: '13.235.0.204:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'hotstar', version: '1.0.0-SNAPSHOT'
            }
        }
        stage('Artifact to S3') {
            steps {
               s3Upload consoleLogLevel: 'INFO', dontSetBuildResultOnFailure: false, dontWaitForConcurrentBuildCompletion: false, entries: [[bucket: 'test-test-hotstar-app', excludedFile: '', flatten: false, gzipFiles: false, keepForever: false, managedArtifacts: false, noUploadOnFailure: false, selectedRegion: 'ap-south-1', showDirectlyInBrowser: false, sourceFile: '**/*.war', storageClass: 'STANDARD_IA', uploadFromSlave: false, useServerSideEncryption: true]], pluginFailureResultConstraint: 'FAILURE', profileName: 's3creds', userMetadata: []            }
        }
    }
}
