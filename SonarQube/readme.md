## installation

**Jenkins**
````
sudo apt update
sudo apt install fontconfig openjdk-21-jre  -y
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y
````
**Docker & maven**
````

sudo apt-get update
sudo apt install maven -y
sudo apt-get install docker.io -y
sudo systemctl start docker
sudo usermod -aG docker ubuntu
sudo usermod -aG docker jenkins
newgrp docker
sudo chmod 777 /var/run/docker.sock
````
**SonarQube**
````
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
````
## Connect to Jenkins 
## Step6: Install Required Plugins:
   **Install below plugins**


````
SonarQube Scanner
````


````
stage view
````
## Install  Tools: Manage Jenkins->Tools
   - add SonarQube Scanner: "sonar-scanner"


### **Configure SonarQube-Scanner and Docker in Global Tool Configuration**

#### SonarQube-Scanner
![image](https://github.com/user-attachments/assets/24589963-9a7e-4d6a-9598-66580c195e30)


#### Docker
![image](https://github.com/user-attachments/assets/289c2e2a-df33-476b-a195-d584db3ef03e)


## Connect to SonarQube
## Log in to Sonarqube and generate token
 - username: admin
 - password: admin
<img width="1902" height="957" alt="image" src="https://github.com/user-attachments/assets/36620768-5f81-440c-b31b-ecf29c609f64" />   


     **SonarQube**
  - Go to  "Manage Jenkins" → Credentials."
  - Click on "Global."
  - Click on "Add Credentials" 
  - Choose "secret text" as the kind of credentials.
  - Enter your sonarqube token and give the credentials an ID (e.g., "sonar-token").
  - Click "create" to save yourcredentials

<img width="1907" height="846" alt="image" src="https://github.com/user-attachments/assets/bcd447f5-4a49-478d-99d6-1379202f4334" />

  - Admin->my account->security->generate token
![image](https://github.com/user-attachments/assets/26cb309d-aa3c-4a74-873f-9e87b2fcce00)

Step5: In Jenkins
     - Manage Jenkins: Credentials
       - Sonar-Token
       


## Configure Sonar Server: Manage Jenkins->System
   - name: "sonar-server"
   - url:
   - token:
![image](https://github.com/user-attachments/assets/c5d05628-1502-4a92-b722-7ad3eed5d587)

## Restart Jenkins
---

##  Create Pipeline
```groovy
pipeline {
    agent any 


    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('code-pull') {
            steps {
                git branch: 'main', url: 'https://github.com/abhipraydhoble/Project-InsureMe.git'
            }
        }

        stage('code-build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage("code-test-analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=InsureMe \
                        -Dsonar.projectName=InsureMe \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

        stage("code-test-quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }


        stage('code-deploy') {
            steps {
                sh 'docker build -t insure-me .'
                sh 'docker run -itd --name insure-me -p 8089:8081 insure-me'
            }
        }
    }
}

```



docker run -d --name sonar -p 9000:9000 sonarqube:lts-community   ies command ka use kya hai pls explain
Ye command Docker ki help se SonarQube ko container ke andar run karne ke liye use hoti hai. SonarQube ka use hum apne code ki quality check karne ke liye karte hain.

1. Command ko samjho
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

Is command mein 5 important parts hain.

1. docker run

Docker ko bolta hai ki ek naya container create karo aur use start karo.

Simple Hindi: Container banao aur chalao.

2. -d

Detached mode — container background mein run hoga. Terminal ko continuously busy nahi rakhega.

Simple Hindi: Background mein chalao.

3. --name sonar

Container ka naam sonar rakhna hai.

Isse baad mein container ko easily identify kar sakte hain.

4. -p 9000:9000

Port mapping karta hai: HOST_PORT:CONTAINER_PORT.

Pehla 9000 — EC2/server ka port.

Doosra 9000 — container ke andar SonarQube ka port.

Isse browser se SonarQube access kar sakte hain.

5. sonarqube:lts-community

Ye Docker image aur uska tag hai.

sonarqube — image ka naam.

lts-community — Long-Term Support Community Edition ka tag.

Agar image local machine par nahi hai, to Docker use registry se pull karne ki koshish karega.

2. Is command ke baad kya hoga?
Browser

http://EC2-PUBLIC-IP:9000

EC2 Server

Port 9000

SonarQube Container

Port 9000

Agar EC2 par chala rahe ho, to Security Group mein TCP port 9000 ko apne trusted IP se allow karna hoga. SonarQube start hone mein kuch time lag sakta hai.

3. Run karne ke baad kaise check karein?
Container running hai ya nahi:

docker ps

SonarQube ke logs dekhne ke liye:

docker logs -f sonar

Browser mein open karo:

http://EC2-PUBLIC-IP:9000
4. Jenkins aur SonarQube ka connection kya hai?
DevOps project mein generally flow aisa hota hai:

Developer code push karta hai — GitHub
Jenkins build karta hai — Maven
SonarQube

Code quality check karta hai

Aage deployment — Docker / AWS
Interview mein simple answer:

“Docker run command is used to start SonarQube in a container. The -d flag runs it in the background, --name assigns a container name, and -p 9000:9000 maps the host port to the container port.”

Yaad rakho: Sirf is command ko run karne se Jenkins ke saath SonarQube integrate nahi hota. Uske liye Jenkins mein SonarQube configuration aur pipeline mein code analysis step bhi add karna hota hai.

i means mujhe ye samaj naji aa raha hai pehle hum docker file bnate phir docker file se image bnate hai to ye direct kese ye samaj nhaimaa raah ahai bus
Aapka confusion bilkul sahi hai! 😊 Aap soch rahe ho ki pehle Dockerfile banate hain, phir image banate hain, phir container run karte hain. To SonarQube mein directly docker run kaise kar diya?

Chalo ekdum simple language mein samajhte hain.

1. Normal Docker process
Maan lo aap apni Java application ko Docker mein chalana chahte ho.

Step 1: Dockerfile banao

Application ko kaise package karna hai, uski instructions

Step 2: Docker image banao


Step 3: Container run karo


Ye process tab use hota hai jab aapko apni requirements ke according image build karni ho.

2. Lekin SonarQube mein kya hua?
SonarQube ki image pehle se bani hui hai. Kisi ne uski Docker image bana kar Docker registry par upload kar di hai.

Isliye aapko khud Dockerfile banane ki zaroorat nahi hai.

SonarQube ki ready-made image

Docker Hub par available

docker run command

Image se container create aur start hota hai

Aapki command:

docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

Docker kya karega?

Check karega ki sonarqube:lts-community image local machine par hai ya nahi.

Agar nahi hai, to registry se image pull karega.

Us image se sonar naam ka container banayega.

Container ko background mein start karega.

Important: Agar aapki machine par image pehle se nahi hai, tab bhi docker run use kar sakte ho. Docker automatically image pull karne ki koshish karta hai.

3. Bas ye difference yaad rakho
Apni application

SonarQube

Dockerfile khud bana sakte ho

Ready-made image available hai

docker build se image banate ho

Image pehle se bani hui hai

docker run se container chalate ho

docker run se container chalate ho

Ek line mein: docker build image banata hai, jabki docker run image se container banakar start karta hai. Ready-made image use kar rahe ho, to docker build karna zaroori nahi hai.

2. Lekin SonarQube mein kya hua?
SonarQube ki image pehle se bani hui hai. Kisi ne uski Docker image bana kar Docker registry par upload kar di hai.   kine raki mai maine to nahi rakhi thi toh docker registry pe kisne rakhi hai

Bahut achha question! Aapne khud image nahi banayi, phir bhi aap use kar paa rahe ho. Chaliye samajhte hain image kisne banayi aur Docker Hub par kisne rakhi.

1. Image kisne banayi?
SonarSource naam ki company SonarQube banati hai aur uski official Docker image maintain karti hai. Us image ko Docker Hub par publish kiya jata hai. 

Docker Hub

SonarSource company

SonarQube software aur official image maintain karti hai

Docker Hub

Online registry jahan image available hai

SonarQube official image dekhein

Aapka EC2 server

Docker image download karta hai aur container chalata hai

2. Aapko Dockerfile banane ki zaroorat kyun nahi padi?
Aapne ye command chalayi:

docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

Ismein sonarqube:lts-community Docker ko batata hai ki kaunsi image use karni hai.

Docker aapke server par image check karta hai. Agar image nahi milti aur registry par available hai, to Docker use automatically pull karne ki koshish karta hai aur usse container start karta hai.

3. Ek simple example
Maan lo aapko ghar banana hai:

Khud ka ghar: Aapko design aur construction karna padega. Ye Dockerfile se apni image banane jaisa hai.

Pehle se bana ghar: Aap use lekar rehna shuru kar sakte ho. Ye ready-made SonarQube image se container chalane jaisa hai.

Bas ek line yaad rakho: Har baar Docker image khud banani zaroori nahi hai. Aap Docker Hub se kisi software ki ready-made image lekar bhi container chala sakte ho.

sonarqube:lts-community ye samaj nagi aaya ye kaha se aay ahai psl
Koi baat nahi 😊 Chalo bilkul basic se samajhte hain ki sonarqube:lts-community kahan se aaya aur humein ise kaise pata chala.

1. Ye naam aapko khud se banane ki zaroorat nahi hai
Aapki command hai:

docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

Iske last mein likha hai:

sonarqube:lts-community

Ye Docker image ka naam aur tag hai, jo Docker Hub par available image ko identify karta hai.

sonarqube = image ka naam.

lts-community = image ka tag, jo specific version/channel ko identify karta hai.

2. Aapko ye naam kahan milega?
Official Docker Hub page:

Docker Hub — SonarQube image tags par jao. Wahan image ke available names aur tags milte hain. 

Docker Hub
+1

3. Aap apne server par khud check kar sakte ho
Image download karne ke liye:

docker pull sonarqube:lts-community

Phir image check karo:

docker images

Dhyan do: Agar tag ab available nahi hai, to pull command error de sakti hai. Docker Hub par currently listed tag ko select karna hoga.

Bas itna samjho: Image ka naam aapne invent nahi kiya. Aapne Docker Hub se milne wale image name ko command mein likha hai. Aise hi Nginx, Jenkins aur MySQL ki ready-made images bhi milti hain.

