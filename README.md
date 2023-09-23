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

Then create a GitHub repo and using below link to push the java app codes.

        https://github.com/kohlidevops/java-app.git

Now Create one project item with pipeline in Jenkins UI

<img width="767" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/154fd0b5-9e80-4358-bda8-7d5d75b8bc7a">

Using pipeline syntax to create a first stage that is Git Checkout.

<img width="793" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/8596a92b-ca14-4b58-90d1-d5e96d0ab742">

After generating the scripts

        git branch: 'main', url: 'https://github.com/kohlidevops/java-app.git'

Lets create a Jenkinsfile in your project repo in local system.

<img width="519" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/1cb023a1-7a42-4beb-88f3-67cbe00ed30e">

Jenkins file for Git Checkout

        pipeline{
	        agent any
	        stages{
		        stage('Git Checkout'){
			        steps{
				        script{
					        git branch: 'main', url: 'https://github.com/kohlidevops/java-app.git'
				        }
			        }
		        }
	        }
        }

Select your project in Jenkins UI and select the Pipeline 

<img width="866" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/bac95f3c-bc21-4e05-8297-542841435269">

<img width="715" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/ce6be341-baf9-4390-9a4a-217353e8016d">

Then Apply and save - To start the build.

<img width="851" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/da0a6ca3-7dda-460a-afe8-0f1f6c586fa9">

Yes! The build has been succeeded. Just have a look at the Jenkins server workspace.

<img width="442" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/cb8bbeab-df4e-4ad6-8a57-8901fcaf7db6">

Now, I'm going to use Jenkins Shared Library which is used to configure Git Check and all the stages and this one is reusable.

Let's create one repo for jekins shared library and clone this repo in locla system. After cloning, create one folder known as vars. Inside the vars folder create one file called as gitCheckout.groovy.

		vi gitCheckout.groovy

  		def call(Map stageParams) {
			checkout([
				$class: 'GitSCM',
				branches: [[ name: stageParams.branch ]],
				userRemoteConfigs: [[ url: stageParams.url]]
			])
		}

  save and exit. Then push the code to repository.
  
<img width="563" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/f47f561e-c5b4-47af-9d68-ee713a5ed980">

As of now, just configure the jenkins shared library repo URL to jenkins UI.

Navigate to Jenkins - Manage Jenkins - Configure system - Global pipeline libraries.

<img width="920" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/4b96cfe8-0ed9-4058-a008-f6fc9d839918">

<img width="839" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/7a53632c-7ce6-40fd-801f-210058c0f731">

Then Apply and save. Now back to Jenkinsfile and update the code.

		pipeline{
			agent any
			stages{
				stage('Git Checkout'){
					steps{
						script{
							gitCheckout{
								branch: "main"
								url: "https://github.com/kohlidevops/java-app.git"
							}
						}
					}
				}
			}
		}

  save and exit - Then puch this code to your java-app.
  
