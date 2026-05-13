# devops
# DEVOPS LAB CHEATSHEET

# PROGRAM 6 — Jenkins CI Pipeline with Maven + GitHub

## Check Java

```bash
java -version
```

---

## Install Maven (Windows)

Download:
[https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi)

Extract ZIP to:

```text
C:\apache-maven
```

CMD:

```bash
set PATH=C:\apache-maven\bin;%PATH%
```

Check:

```bash
mvn -version
```

---

## Install Git

Download:
[https://git-scm.com/download/win](https://git-scm.com/download/win)

Check:

```bash
git --version
```

---

## Install Jenkins

Download:
[https://www.jenkins.io/download/](https://www.jenkins.io/download/)

Open:

```text
http://localhost:8080
```

Get password:

```bash
type "C:\Program Files\Jenkins\secrets\initialAdminPassword"
```

Install Suggested Plugins.

---

# Jenkins Tool Configuration

Go:

```text
Manage Jenkins → Tools
```

Add:

## JDK

```text
Name: JDK17
JAVA_HOME:
C:\Program Files\Java\jdk-17
```

## Maven

```text
Name: Maven
MAVEN_HOME:
C:\apache-maven
```

## Git

```text
Path to Git:
C:\Program Files\Git\bin\git.exe
```

Save.

---

## Create Maven Project

Git Bash:

```bash
mkdir exp6
cd exp6
```

```bash
mvn archetype:generate -DgroupId=com.example -DartifactId=calculator -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

```bash
cd calculator
```

Build:

```bash
mvn package
```

---

## Modify pom.xml

Open:

```bash
notepad pom.xml
```

Add AFTER `</dependencies>`:

```xml
<build>
  <plugins>

   <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.11.0</version>
   </plugin>

   <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.1.2</version>
   </plugin>

  </plugins>
</build>
```

---

## Push to GitHub

Create GitHub repo:

```text
exp6
```

Git Bash:

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/USERNAME/exp6.git
git push -u origin main
```

---

## Jenkins Pipeline Script

```groovy
pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'JDK17'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/USERNAME/exp6.git'
            }
        }

        stage('Build') {
            steps {
                bat 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                bat 'mvn test'
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true,
            testResults: '**/target/surefire-reports/*.xml'
        }
    }
}
```

---

# PROGRAM 7 — Ansible Basics

## Install Ansible (Ubuntu)

```bash
sudo apt update
sudo apt install ansible -y
```

Check:

```bash
ansible --version
```

---

## Create hosts.ini

```bash
gedit hosts.ini
```

Paste:

```ini
[local]
localhost ansible_connection=local
```

---

## Create setup.yml

```bash
gedit setup.yml
```

Paste:

```yaml
---
- name: Message Playbook
  hosts: local
  gather_facts: no

  tasks:
    - name: Print Hello Message
      debug:
        msg: "Hello from Ansible"
```

---

## Run Playbook

```bash
ansible-playbook -i hosts.ini setup.yml
```

---

# PROGRAM 8 — Jenkins + Maven + Ansible

## Install Maven (Ubuntu)

```bash
sudo apt update
sudo apt install maven -y
```

Check:

```bash
mvn -version
```

---

## Create Maven Project

```bash
mkdir hello
cd hello
```

```bash
mvn archetype:generate -DgroupId=com.example -DartifactId=hello-world -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

```bash
cd hello-world
```

Build:

```bash
mvn clean package
```

---

## Install Jenkins (Ubuntu)

Download:

```bash
wget https://get.jenkins.io/war-stable/latest/jenkins.war
```

Run Jenkins:

```bash
java -jar jenkins.war --httpPort=9090
```

Open:

```text
http://localhost:9090
```

Get password:

```bash
cat ~/.jenkins/secrets/initialAdminPassword
```

---

## Install Ansible

```bash
sudo apt install ansible -y
```

---

## Create hosts.ini

```bash
gedit ~/hosts.ini
```

Paste:

```ini
[local]
localhost ansible_connection=local
```

---

## Create setup.yml

```bash
gedit ~/setup.yml
```

Paste:

```yaml
- name: Deploy Maven Artifact
  hosts: local

  tasks:

    - name: Create deploy directory
      file:
        path: /home/student/deploy
        state: directory

    - name: Copy JAR
      copy:
        src: /home/student/Desktop/hello-world/target/hello-world-1.0-SNAPSHOT.jar
        dest: /home/student/deploy/
```

---

## Jenkins Pipeline Script

```groovy
pipeline {
    agent any

    stages {

        stage('Build') {
            steps {
                sh 'cd /home/student/Desktop/hello-world && mvn clean package'
            }
        }

        stage('Deploy') {
            steps {
                sh 'ansible-playbook -i /home/student/hosts.ini /home/student/setup.yml'
            }
        }
    }
}
```

---

# PROGRAM 10 — Azure Pipelines + Maven + GitHub

## Install Maven

```bash
sudo apt update
sudo apt install maven -y
```

Check:

```bash
mvn -version
```

---

## Create Maven Project

```bash
mkdir maventest1
cd maventest1
```

```bash
mvn archetype:generate -DgroupId=com.dineshonjava -DartifactId=Javateam -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

```bash
cd Javateam
mvn test
```

---

## Push to GitHub

Create GitHub repo:

```text
maventestazure
```

Git Bash:

```bash
git init
git add .
git commit -m "azure pipeline example"
git branch -M main
git remote add origin https://github.com/USERNAME/maventestazure.git
git push -u origin main
```

---

## Azure DevOps

Open:
[https://dev.azure.com](https://dev.azure.com)

Create:

* Organization
* Public Project

Enable:

```text
Allow Public Projects
```

---

## Create Pipeline

Go:

```text
Pipelines → Create Pipeline
```

Choose:

```text
GitHub
```

Select repo:

```text
maventestazure
```

Choose:

```text
Maven Template
```

Save and Run.

---

## If Pipeline Fails

Edit YAML:

```yaml
trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
- task: Maven@4
  inputs:
    mavenPomFile: 'pom.xml'
    goals: 'package'
```

Run again.

SUCCESS:

```text
Pipeline succeeded
```
