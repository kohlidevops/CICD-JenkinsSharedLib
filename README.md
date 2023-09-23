# CICD-JenkinsSharedLib
CICD with Jenkins Shared Library, Sonar, Docker, ECR and Kubernetes

Step -1: Launch Jenkins Instance

I'm going to use SonarQube in Jenkins Server, So I will choose the T3.medium instance type in AWS Cloud.

SSH to mahcine and install the jenkins using below commands.

    #!/bin/bash
    sudo apt update -y
    sudo apt upgrade -y 
    sudo apt install openjdk-17-jre -y
    curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
      /usr/share/keyrings/jenkins-keyring.asc > /dev/null
      echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
      https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
      /etc/apt/sources.list.d/jenkins.list > /dev/null
    sudo apt-get update -y 
    sudo apt-get install jenkins -y

I will be create a SonarQube and run as docker container. So first, I'm going to install a docker using below commands.

        #!/bin/bash
        sudo apt update -y
        sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
        sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable" -y
        sudo apt update -y
        apt-cache policy docker-ce -y
        sudo apt install docker-ce -y
        #sudo systemctl status docker
        sudo chmod 777 /var/run/docker.sock

Then run below command to run a SonarQube as docker container

        docker run -d --name sonarqube -p 9000:9000 -p 9092:9092 sonarqube

That's it.

Now you can able to access the Jenkins console in browser. Then Unlock the Jenkins.

<img width="865" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/30fe5f40-f46c-466c-a544-6d586c987788">

Then Install suggested plugins and Create one user and go ahead.

<img width="834" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/0125079e-c9cc-430f-8846-01cc4e2a02aa">

Jenkins installation has been completed. Now SSH to machine and check the SonarQube container is running or not.

        $docker container ls

Yup! Its up and running with host port 9000.

<img width="897" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/277c2d20-4ebe-4f7f-8017-d3e6bca76a5d">

Now I'm going to configure SonarQube in browser.

<img width="631" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/37dbdf7c-40fb-462e-a216-b0f63e9079ac">

Default user name is admin and password is admin. Then change the password as you want.

<img width="929" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/7c7bdec9-9066-4c7f-8590-2a63fabb91dd">



