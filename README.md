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
								branch: "main",
								url: "https://github.com/kohlidevops/java-app.git"
							}
						}
					}
				}
			}
		}

  save and exit - Then puch this code to your java-app.

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

  		@Library('my-shared-lib') _
		pipeline{
			agent any
			stages{
				stage('Git Checkout'){
					steps{
						script{
							gitCheckout{
								branch: "main",
								url: "https://github.com/kohlidevops/java-app.git"
							}
						}
					}
				}
			}
		}

save and exit - Then puch this code to your java-app. Remember - "@Library('my-shared-lib') _" Its refer the name which is declared in Jenkins - Manage Jenkins - system - Global Pipeline Libraries

<img width="545" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/35c3a3c0-2070-4d92-80df-1ece873d14aa">

I just create a PAC in GitHub repository - GitHub - Profile - Settings - Developer Settings - Generate PAC with expiration. Then copy paste the token in Jenkins - Manage Jenkins -System - Global pipeline libraries - add this token with GitHub URL to avoid below error.

<img width="671" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/00591e95-9461-41a0-9ad7-39c953bae059">

Like below, to add the PAC.

<img width="632" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/2e30eeb9-23f7-4474-95ed-fe1efb9cd2ea">

Now start the build - Its succeeded - So its working perfect with Jenkins shared library.

<img width="841" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/ddd75c49-8ad4-43d4-87d7-a3de7c49d0db">

Step -2: Create a Unit Testing stage

First install the maven in Jenkins server to unit test the code using below commands.

		sudo apt update -y
		sudo apt install maven -y
		mvn -version
  
Go to your local system and select jenkins shared library repo and create one file called as mvnTest.groovy

		def call() {
			sh 'mvn test'
			}

save and exit - Then push the code to repo.

<img width="468" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/236814ff-522d-483b-b6e6-3eb3209f75e9">

To update the Unit Test staging in Jenkins file

		@Library('my-shared-lib') _

		pipeline{
        		agent any
        		stages{
                		stage('Git Checkout'){
                        		steps{
                        		gitCheckout(
                        			branch: "main",
                            			url: "https://github.com/kohlidevops/java-app.git"
                                        	)
                        		}
                		}
                		stage('Unit Test with Maven'){
                			steps{
                				script{
                					mvnTest()
                				}
                			}
                		}
        		}
		}

  save and exit - Then push the code to java-app repo. Then start the build. 

  It may this build get failed due to java version. 

##  If any facing issue with - maven (Failed to execute goal  [32morg.apache.maven.plugins:maven-compiler-plugin:3.8.1:compile [m  [1m(default-compile) [m on project  [36mkubernetes-configmap-reload [m:  [1;31mFatal error compiling [m: java.lang.ExceptionInInitializerError:), then follow command to resolve the issues.

		First u check your java version or mvn -v - it will list also java version
		$mvn -v
		If java version is 17, then install java 11
		$sudo apt install openjdk-11-jre -y

		To change the default java version
		$sudo update-alternatives --config java

		Then check java version
		$java -version

now you start the build. It should success. It seems version dependecies.
  
Step -3: To configure the Maven Integration Testing stage

Go to local systems and navigate to jenkins shared library repo and inside the vars directory create one file is called as mvnIntegrationTest.groovy

		def call(){
			sh 'mvn verify -DskipUnitTests'
		}

save and exit - Then push the code to remote repo.

Then update the Jenkins file to call mvnIntegrationTest.groovy

		@Library('my-shared-lib') _

		pipeline{
        		agent any
        		stages{
                		stage('Git Checkout'){
                        		steps{
                        		gitCheckout(
                        			branch: "main",
                            			url: "https://github.com/kohlidevops/java-app.git"
                                        	)
                        		}
                		}
                	stage('Unit Test with Maven'){
                		steps{
                			script{
                				mvnTest()
                			}
                		}
                	}
                	stage('Integration Test with Maven'){
                		steps{
                			script{
                				mvnIntegrationTest()
                				}
                			}
                		}                
        		}
		}

save and exit - Then push the code to java-app repo. Then start the build.

<img width="768" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/07170d11-0729-463e-9656-a4127513133d">

The build has been succeeded with maven integration testing stage.

Step -4: Configure Condition Stage

Now I'm going to add "Condition Stage" for Jenkins. This condition stage will run the particular stage based on user choice.

Just update the Jenkinsfile like below

		@Library('my-shared-lib') _

		pipeline{
        		agent any

        		parameters{
        			choice(name: 'action', choices: 'create\ndelete', description: 'Choose create/Destroy')
        		}
        		stages{
                		stage('Git Checkout'){
                				when { expression {  params.action == 'create' } }
                        		steps{
                        		gitCheckout(
                        			branch: "main",
                            			url: "https://github.com/kohlidevops/java-app.git"
                                        		)
                        		}
                		}
                		stage('Unit Test with Maven'){
                				when { expression {  params.action == 'create' } }
                			steps{
                				script{
                					mvnTest()
                					}
                				}
                			}
                		stage('Integration Test with Maven'){
                				when { expression {  params.action == 'create' } }
                			steps{
                				script{
                					mvnIntegrationTest()
                					}
                				}
                			}                
        			}
			}

Save and exit - Then start the build. - Working fine.

Step -5: Generate Token in SonarQube console

Login to SonarQube console - select - Administration - Security - Users - Administrator - create a token

<img width="879" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/144ee0e9-4f36-49b6-b86e-d51d118e483f">

copy it and will use later.

Step -6: Configure SonarQube in Jenkins

Jenkins - Manage Jenkins - Plugins - Install below plugins

		SonarQube Scanner
  		Sonar Gerrit
    		SonarQube Generic Coverage
      		Sonar Quality Gates
		Quality Gates

Then install without restart.

Jenkins - Manage Jenkins - System - Sonar Qube Servers

<img width="725" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/51fcff8e-9131-4ebe-9885-e3ab64d4a2b5">

Apply and save.

Again Jenkins - Manage Jenkins - System - SonarQube servers

<img width="747" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/1e627d63-cbe3-4e69-bac6-d6264e553660">

Copy paste the Token ID in Secret text then save.

<img width="700" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/04f1f85e-a718-44d7-aca0-2623d61d5d9a">

Then select the authentication token from the drop down list.

<img width="773" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/b5a28273-e359-444a-89c4-a1bd3cd960b9">

Then Apply & Save.

Step -7: Create Static Code Analysis stage with SonarQube

Go to pipeline syntax and create a syntax for sonar-api token

<img width="822" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/fbf0e147-c29f-4723-aac1-5e0f61130fb7">

Navigate to local system - jenkins shared lib repo - inside vars folder create a file called as statiCodeAnalysis.groovy

		def call(){
       			 withSonarQubeEnv(credentialsId: 'sonar-api') {
                		sh 'mvn clean package sonar:sonar'
                                                }
        		}


save & exit - Then push the code to remote repo.

To update the Jenkinsfile for StatiCodeAnalysis

@Library('my-shared-lib') _

pipeline{
        agent any

        parameters{
        	choice(name: 'action', choices: 'create\ndelete', description: 'Choose create/Destroy')
        }
        stages{
                stage('Git Checkout'){
                			when { expression {  params.action == 'create' } }
                        steps{
                        gitCheckout(
                        	branch: "main",
                            url: "https://github.com/kohlidevops/java-app.git"
                                        )
                        }
                }
                stage('Unit Test with Maven'){
                			when { expression {  params.action == 'create' } }
                		steps{
                			script{
                			mvnTest()
                			}
                		}
                	}
                stage('Integration Test with Maven'){
                			when { expression {  params.action == 'create' } }
                		steps{
                			script{
                				mvnIntegrationTest()
                			}
                		}
                	}
                 stage('Static Code Analysis with SonarQube'){
                                        when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                                statiCodeAnalysis()
                                        }
                                }
                        }                
        	}
	}

save and exit - Then push the code to remote repo. Then start the build with parameters.

<img width="692" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/f7cce9a6-328a-4779-8269-c5841e2c7867">

Yes build has been succeeded.

<img width="890" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/49f7b471-23dc-4703-b12d-efd4d6ffe456">

You can check with SonarQube console to see the results

<img width="943" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/1493a0b7-38ec-4acb-b922-5d403f6277cb">

Step -7: Create a SonarQube Webhook

Navigate to SonarQube console - Administration - Webhook - Create

<img width="791" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/33260f78-4488-40d0-916e-aa10de1d07c3">

Create a file called as QualityGateStatus.groovy inside a var folder in jenkins shared lib folder

	def call(){
        	waitForQualityGate abortPipeline: false, credentialsId: 'sonar-api'
        	}

save and exit the code - Then push this code to jenkins shared lib repo.

Now Update the Jenkinsfile with QualityGateStatus stage

		@Library('my-shared-lib') _

		pipeline{
        		agent any

        		parameters{
                		choice(name: 'action', choices: 'create\ndelete', description: 'Choose create/Destroy')
        		}
        		stages{
                		stage('Git Checkout'){
                               	        when { expression {  params.action == 'create' } }
                        		steps{
                        			gitCheckout(
                                			branch: "main",
                            				url: "https://github.com/kohlidevops/java-app.git"
                                        		)
                       	 			}
                		}
                	stage('Unit Test with Maven'){
                        	               when { expression {  params.action == 'create' } }
                                	steps{
                                        	script{
                                        		mvnTest()
                                        		}
                                		}
                        		}
                	stage('Integration Test with Maven'){
                        	                when { expression {  params.action == 'create' } }
                                	steps{
                                        	script{
                                                	mvnIntegrationTest()
                                        		}
                                		}
                        		}
                	stage('Static Code Analysis with SonarQube'){
                         	               when { expression {  params.action == 'create' } }
                                	steps{
                                        	script{
                                                	statiCodeAnalysis()
                                        		}
                                		}
                        		}
                	stage('Code Quality Status Check with SonarQube'){
                         	               when { expression {  params.action == 'create' } }
                                	steps{
                                        	script{
                                        		QualityGateStatus()
                                        	}
                                	}
                        	}
                	}
        	}

save and exit - Then push the code to repo. Then start the build.

Yes build has been succeeded. 

<img width="500" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/eea0b4dc-8f1a-4802-ab22-cafaff0288cc">

<img width="932" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/393dd7d5-4f10-48ed-ac19-ba5a08eac347">

Step -8: Create a Maven Build Stage

Navigate to local system - jenkins shared lib repo - inside vars folder create a file called as mvnBuild.groovy.

		def call(){
			sh 'mvn clean install'
			}

save and exit - Then push the code to jenkins shared lib repo

To update the Jenkinsfile for mvnBuild stage

		@Library('my-shared-lib') _

		pipeline{
        		agent any
        		parameters{
                		choice(name: 'action', choices: 'create\ndelete', description: 'Choose create/Destroy')
        		}
        		stages{
                		stage('Git Checkout'){
                                 	       when { expression {  params.action == 'create' } }
                        		steps{
                        			gitCheckout(
                                			branch: "main",
                            				url: "https://github.com/kohlidevops/java-app.git"
                                        		)
                        		}
                		}
                	stage('Unit Test with Maven'){
                                        when { expression {  params.action == 'create' } }
                                	steps{
                                        	script{
                                        		mvnTest()
                                        		}
                                		}
                        		}
                	stage('Integration Test with Maven'){
                         	        when { expression {  params.action == 'create' } }
                                	steps{
                                        	script{
                                                	mvnIntegrationTest()
                                        		}
                                		}
                        		}
                	stage('Static Code Analysis with SonarQube'){
                                        when { expression {  params.action == 'create' } }
                                	steps{
                                        	script{
                                                	statiCodeAnalysis()
                                        	}
                                	}
                        	}
                	stage('Code Quality Status Check with SonarQube'){
                                        when { expression {  params.action == 'create' } }
                                	steps{
                                        	script{
                                        		QualityGateStatus()
                                        	}
                                	}
                        	}
                	stage('Maven Build Stage'){
                                        when { expression {  params.action == 'create' } }
                                	steps{
                                        	script{
                                        		mvnBuild()
                                        	}
                                	}
                        	}
                	}
        	}

save and exit - Then push the code to java app repo. Then start the build in Jenkins.

<img width="944" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/5bf3562e-c69f-4eda-b9d7-f447f74c355b">

Perfect! The Maven BUild has been succeeded and artifact (jar) file is created. You can check the Jar file 

Select - Your last Build - workspace - click link - target folder - .jar file is available.

<img width="868" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/55e8404c-2c8e-4cd7-8797-40f191f66948">

Step -9: Create a Dockerfile

Create this Dockerfile in your java-app root folder.

To create a Dockerfile to copy a jar file from target directory to local folder app

		FROM openjdk:8-jdk-alpine
		WORKDIR /app
		COPY ./target/*.jar /app.jar
		CMD ["java", "-jar", "app.jar"]

save abd exit - Then push this docker file to java-app repo.

Step -10: To create a docker build stage

Navigate to local system - jenkins shared lib repo - inside vars folder create a file called as dockerBuild.groovy.

		def call(){
        		sh """
        		docker image build -t latchudevops/javapp .
        		docker image tag latchudevops/javapp latchudevops/javapp:v1
        		docker image tag latchudevops/javapp latchudevops/javapp:latest
        		"""
			}

save and exit - Then push this code to jenkins shared lib repo.

Now update the Jenkinsfile to call dockerBuild stage

		@Library('my-shared-lib') _

		pipeline{
        		agent any
        			parameters{
                			choice(name: 'action', choices: 'create\ndelete', description: 'Choose create/Destroy')
                  			}
        		stages{
                		stage('Git Checkout'){
                                        when { expression {  params.action == 'create' } }
                        	steps{
                        		gitCheckout(
                                		branch: "main",
                            			url: "https://github.com/kohlidevops/java-app.git"
                                        		)
                        		}
                		}
                	stage('Unit Test with Maven'){
                                        when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                        mvnTest()
                                        }
                                }
                        }
                	stage('Integration Test with Maven'){
                                        when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                                mvnIntegrationTest()
                                        }
                                }
                        }
                	stage('Static Code Analysis with SonarQube'){
                                        when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                                statiCodeAnalysis()
                                        }
                                }
                        }
                	stage('Code Quality Status Check with SonarQube'){
                                        when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                                QualityGateStatus()
                                        }
                                }
                        }
                	stage('Maven Build Stage'){
                                        when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                                mvnBuild()
                                        }
                                }
                        }
                	stage('Docker Image Build'){
                                        when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                                dockerBuild()
                                        }
                                }
                        	}
                	}
       	 	}

save and exit - Then push this code to java repo. Then start the build.

<img width="945" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/1e88cec1-435c-48b7-9b48-8da4684eb072">

Yes, Build has been succeeded and you can check the docker images in Jenkins server.

<img width="645" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/6c04f411-d719-49b2-9eec-781fa8ef4f98">

Step -11: To Create a Scan Docker Image stage

First need to install a Trivy app to scan a Docker image in Jenkins Server. Trivy is a Simple and Comprehensive Vulnerability Scanner for Containers and other Artifacts and it is Suitable for Continuous Integration and Continuous Deployment.

SSH to Jenkins server and install below commands.

		#!/bin/bash
		sudo apt-get install wget apt-transport-https gnupg lsb-release
		wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
		echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
		sudo apt-get update
		sudo apt-get install trivy
  		trivy

To Create a dockerScanImage.groovy file

Navigate to local system - jenkins shared lib repo - inside vars folder create a file called as dockerScanImage.groovy.

		def call(){
			sh """
        		trivy image latchudevops/javapp:latest > scan.txt
        		cat scan.txt
        		"""
		}

save and exit - Then push this code to jenkins shared lib

Now create a Jenkinsfile for dockerImageScan call

		@Library('my-shared-lib') _

		pipeline{
        	agent any
        	parameters{
                	choice(name: 'action', choices: 'create\ndelete', description: 'Choose create/Destroy')
        		}
        	stages{
                	stage('Git Checkout'){
                                        when { expression {  params.action == 'create' } }
                        	steps{
                        		gitCheckout(
                                		branch: "main",
                            			url: "https://github.com/kohlidevops/java-app.git"
                                        	)
                        		}	
                		}
                stage('Unit Test with Maven'){
                                        when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                    	                    mvnTest()
                         	               }
                                	}
                        	}
                stage('Integration Test with Maven'){
                                        when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                                mvnIntegrationTest()
                               		         }
                                	}
                        	}
                stage('Static Code Analysis with SonarQube'){
                                        when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                                statiCodeAnalysis()
                                	        }
                                	}
                        	}
                stage('Code Quality Status Check with SonarQube'){
                                        when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                        	QualityGateStatus()
                                        	}
                                	}
                        	}
                stage('Maven Build Stage'){
                                        when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                        	mvnBuild()
                                        	}
                                	}
                        	}
                stage('Docker Image Build'){
         				when { expression {  params.action == 'create' } }
            			steps{
               				script{
                   				dockerBuild()
               					}
            				}
        			}
        	stage('Docker Image Scanning'){
         				when { expression {  params.action == 'create' } }
            			steps{
               				script{
                				dockerImageScan()
               					}
            				}
        			}
            		}
        	}

Save and exit - Then push the code to java app repo. Then start the build.

<img width="953" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/98b9ed45-2256-4550-8974-69d7f6e33c5f">

The build has been succeeded and you can see the scanning vulnerability in Jenkins build console output.

<img width="911" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/db446d2a-b107-4bc7-aa69-70d7cb4d9d0f">

Step -12: To Create a Docker Image Push Stage

Go to Jenkins console - pipeline syntax

<img width="711" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/70c5df3b-876f-46aa-903a-19d2bd5e0de1">

select - username and password - Add Jenkins

<img width="655" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/f2ee99cd-2ae9-48f5-9964-0c6a1eb2c10a">

<img width="710" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/95ec9b2e-75c1-40f1-95a7-b02809a8580a">

Provide user and password vailable

<img width="697" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/09f4d026-748d-42f3-a601-a16059a3e81e">

Now generate a Syntax.

To Create a dockerImagePush.groovy file

Navigate to local system - jenkins shared lib repo - inside vars folder create a file called as dockerImagePush.groovy.

		def call(){
        		withCredentials([usernamePassword(
                		credentialsId: 'docker_hub_password',
                		passwordVariable: 'PASSWORD',
                		usernameVariable: 'USER'
        		)]) {
        		sh "docker login -u '$USER' -p '$PASSWORD'"
        			}
        		sh "docker image push latchudevops/javapp:v1"
        		sh "docker image push latchudevops/javapp:latest"
			}

save ans exit - Then push this code to jenkins shared lib repo.

Now update the Jenkinsfile to call dockerImagePush.groovy file

		@Library('my-shared-lib') _
		pipeline{
        	agent any
        		parameters{
                		choice(name: 'action', choices: 'create\ndelete', description: 'Choose create/Destroy')
        			}
        	stages{
                	stage('Git Checkout'){
                                       when { expression {  params.action == 'create' } }
                        steps{
                        	gitCheckout(
                                	branch: "main",
                            		url: "https://github.com/kohlidevops/java-app.git"
                                        )
                        	}
                	}
                stage('Unit Test with Maven'){
                                     when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                        	mvnTest()
                                        }
                                }
                        }
                stage('Integration Test with Maven'){
                                        when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                                mvnIntegrationTest()
                                        }
                                }
                        }
                stage('Static Code Analysis with SonarQube'){
                                        when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                                statiCodeAnalysis()
                                        }
                                }
                        }
                stage('Code Quality Status Check with SonarQube'){
                                        when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                        	QualityGateStatus()
                                        }
                                }
                        }
                stage('Maven Build Stage'){
                                        when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                        	mvnBuild()
                                        }
                                }
                        }
                stage('Docker Image Build'){
         				when { expression {  params.action == 'create' } }
            			steps{
               				script{
                   				dockerBuild()
               					}
            				}
        			}
        	stage('Docker Image Scanning'){
         				when { expression {  params.action == 'create' } }
            			steps{
               				script{
                				dockerImageScan()
               					}
            				}
        			}
        	stage('Docker Image Push'){
         				when { expression {  params.action == 'create' } }
            			steps{
               				script{
                   				dockerImagePush()
               					}
            				}
        			}
            	   	}
        	}
  
save and exit - Then push this code to java app repo

Then start the build and check the docker hub repo whether the image is pushed or not.

<img width="951" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/c01b190e-bf4c-43d7-8d57-ca4a101d3f21">

Build has been succeeded and just have a look at the docker hub repo.

<img width="937" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/d0bf9dfa-049c-44ce-b5ee-1cbd74d6a8d1">

<img width="926" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/cbb0fb18-37bd-4e62-b261-1e459bc2d8f2">

Step -13: Docker Cleanup Stage in Jenkins Server

To Create a dockerImageCleanup.groovy file

Navigate to local system - jenkins shared lib repo - inside vars folder create a file called as dockerImageCleanup.groovy.

		def call(){
        		sh "docker rmi latchudevops/javapp:v1"
        		sh "docker rmi latchudevops/javapp:latest"	
			}

save and exit - Then push this code jenkins shared lib repo.

Now, Update the Jenkinsfile to call dockerImageCleanup.groovy call.

		@Library('my-shared-lib') _
		pipeline{
        	agent any
        		parameters{
                		choice(name: 'action', choices: 'create\ndelete', description: 'Choose create/Destroy')
        			}
        	stages{
                	stage('Git Checkout'){
                                       when { expression {  params.action == 'create' } }
                        steps{
                        	gitCheckout(
                                	branch: "main",
                            		url: "https://github.com/kohlidevops/java-app.git"
                                        )
                        	}
                	}
                stage('Unit Test with Maven'){
                                     when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                        	mvnTest()
                                        }
                                }
                        }
                stage('Integration Test with Maven'){
                                        when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                                mvnIntegrationTest()
                                        }
                                }
                        }
                stage('Static Code Analysis with SonarQube'){
                                        when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                                statiCodeAnalysis()
                                        }
                                }
                        }
                stage('Code Quality Status Check with SonarQube'){
                                        when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                        	QualityGateStatus()
                                        }
                                }
                        }
                stage('Maven Build Stage'){
                                        when { expression {  params.action == 'create' } }
                                steps{
                                        script{
                                        	mvnBuild()
                                        }
                                }
                        }
                stage('Docker Image Build'){
         				when { expression {  params.action == 'create' } }
            			steps{
               				script{
                   				dockerBuild()
               					}
            				}
        			}
        	stage('Docker Image Scanning'){
         				when { expression {  params.action == 'create' } }
            			steps{
               				script{
                				dockerImageScan()
               					}
            				}
        			}
        	stage('Docker Image Push'){
         				when { expression {  params.action == 'create' } }
            			steps{
               				script{
                   				dockerImagePush()
               					}
            				}
        			}
	   	stage('Docker Image Cleanup'){
         				when { expression {  params.action == 'create' } }
            			steps{
               				script{
                   				dockerImageCleanup()
               					}
            				}
        			}
            	   	}
        	}

save and exit - Then push this code to java app repo. Then start the build.

Yes Build has been succeeded. 

Step -13: Docker Image Push to Amazon Elastic Container Registry (ECR)

we can use same code (but upto Maven Build stage) which we used in Jenkinsfile. Now Im going to create Jenkinsfile-ECR in java app repo.

Just use below link to copy/paste Jenkinsfile-ECR 		

		https://github.com/kohlidevops/java-app/blob/main/Jenkinsfile-ECR

save and exit - save as Jenkinsfile-ECR in java app repo.

To Create ECR repo in Amazon

Navigate to AWS console and select - ECR

<img width="622" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/d28d5be2-3056-431b-a76f-aa2f7fa07f54">

just leave others as default.

<img width="789" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/c3dd2749-a8c6-4cdf-b687-567b8b21f610">

ECR has been created.

Step -14: Docker Image Build, Scan and Push in Amazon ECR

To Create a dockerBuildECR.groovy file

Navigate to local system - jenkins shared lib repo - inside vars folder create a file called as dockerBuildECR.groovy.

		def call(){
        		sh """
        		docker build -t latchudevops .
        		docker tag latchudevops:latest 1234567890.dkr.ecr.ap-south-1.amazonaws.com/latchudevops:latest
        		"""
			}

save and exit - Then push this code to jenkins shared lib

Navigate to local system - jenkins shared lib repo - inside vars folder create a file called as dockerImageScanECR.groovy.

		def call(){
        		sh """
        		trivy image 1234567890.dkr.ecr.ap-south-1.amazonaws.com/latchudevops:latest > scan.txt
        		cat scan.txt
        		"""
			}

save and exit - Then push this code to jenkins shared lib

Navigate to local system - jenkins shared lib repo - inside vars folder create a file called as dockerImagePushECR.groovy.

		def call(){
        		sh """
        		aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 1234567890.dkr.ecr.ap-south-1.amazonaws.com
	  		docker push 1234567890.dkr.ecr.ap-south-1.amazonaws.com/latchudevops:latest
        		"""
			}

save and exit - Then push this code to jenkins shared lib

Step -15: To configure AWS in Jenkins

		sudo apt install awscli -y

Create an IAM Role - Trusted Entity as EC2 and Policy could be ECR full access and create IAM ROle

Associate this IAM Role to Jenkins Server

Step -16: To Create a new Pipeline Job in Jenkins

<img width="863" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/f07fcc60-0ed6-4df0-a8ba-8594180fc7b2">

<img width="759" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/d3a7e6fc-a2f8-4192-9277-31500a205b8b">

Apply and save - Then start the build.

Yes, Build has been succeeded.

<img width="949" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/a8eb7614-4b4a-4767-8b1d-b7e995b215ca">

The Images has been pushed to Amazon Elastic Container Registry.

<img width="960" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/89ddd09f-b53a-48f7-bbf5-33143a875dd9">

Step -17: To Create EKS cluster using Terraform module

eks-module available in same repo.

First create IAM Accesskey and secretkey then configure in Jenkins - Manage jenkins - Credentials - Add - secret text - create for both AWS_ACCESS_KEY_ID and AWS_SECRET_KEY_ID.

<img width="869" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/e87b51bc-d982-46b1-9226-fc6d0a3cdfa9">

Then update environment vailable in Jenkinsfile-ECR

		environment{
        		ACCESS_KEY = credentials('AWS_ACCESS_KEY_ID')
        		SECRET_KEY = credentials('AWS_SECRET_KEY_ID')
    			}

save and exit - Then push this code to java app repo. Then start the build.

Please make ensure Terraform has been installed in Jenkins server.

  		https://tecadmin.net/how-to-install-terraform-on-ubuntu/

Please use below link to copy paste the Jenkinsfile-ECR

		https://github.com/kohlidevops/java-app/blob/main/Jenkinsfile-ECR

Build has been succeeded.

<img width="959" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/2e8f705e-544a-47d2-b6c4-2cb08bd7bd9e">

The Elastic Kubernetes Service has been created with Terrafor tool.

<img width="871" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/aca42372-1392-4fe6-b6df-a2f5b55df119">

Step -18: Connect to EKS Cluster Stage

Please use below link to copy paste the Jenkinsfile-ECR

		https://github.com/kohlidevops/java-app/blob/main/Jenkinsfile-ECR

save and exit - Then start the build.

<img width="958" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/a434e008-34ab-4a11-8aa1-dba06d9bde26">

The build has been succeeded.

<img width="753" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/a92f0e27-14d5-4707-a494-eee727ae4341">

Step -19: Deploy on EKS Cluster

Install kubectl in Jenkins Server

		sudo snap install kubectl --classic

Please use below link to copy paste the Jenkinsfile-ECR

save and exit - Then push the code to java app repo. Then start the build

Ready to Deploy Button

<img width="951" alt="image" src="https://github.com/kohlidevops/CICD-JenkinsSharedLib/assets/100069489/9d471bdf-52fc-45ea-ab34-1b48be3aabbc">

