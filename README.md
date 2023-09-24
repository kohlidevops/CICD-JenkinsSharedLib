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

