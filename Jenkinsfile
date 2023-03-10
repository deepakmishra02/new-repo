pipeline {
	agent {	
		label 'pipeline-docker'
		}
	stages {
		stage("SCM") {
			steps {
				git branch: 'main', url: 'https://github.com/deepakmishra02/new-repo.git'
				}
			}

		stage("build") {
			steps {
				sh 'sudo mvn dependency:purge-local-repository'
				sh 'sudo mvn clean package'
				}
			}
		stage("Image") {
			steps {
				sh 'sudo docker build -t java-repo:$BUILD_TAG .'
				sh 'sudo docker tag java-repo:$BUILD_TAG deepakmishra02/pipeline-java:$BUILD_TAG'
				}
			}
				
	
		stage("Docker Hub") {
			steps {
			withCredentials([string(credentialsId: 'docker_hub', variable: 'docker_hub_var')]) {
				sh 'sudo docker login -u deepakmishra02 -p ${docker_hub_var}'
				sh 'sudo docker push deepakmishra02/pipeline-java:$BUILD_TAG'
				}
			}	

		}
		stage("QAT Testing") {
			steps {
			
				sh 'sudo docker run -dit -p 8080:8080  deepakmishra02/pipeline-java:$BUILD_TAG'
				}
			}
		stage("testing website") {
			steps {
				retry(5) {
				sh 'curl --silent http://172.31.12.49:8080/pipeline-java/ | grep -i "india" '
					}
				}
			}

		stage("Approval status") {
			steps {
				script {
					 Boolean userInput = input(id: 'Proceed1', message: 'Promote build?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']])
                echo 'userInput: ' + userInput
					}
				}	
		}
		stage("Prod Env") {
			steps {
			 sshagent(['ubuntu']) {
			    sh 'ssh -o StrictHostKeyChecking=no ubuntu@65.2.140.187 sudo docker rm -f $(sudo docker ps -a -q)' 
	                    sh "ssh -o StrictHostKeyChecking=no ubuntu@65.2.140.187 sudo docker run  -d  -p  49153:8080  srronak/javatest-app:$BUILD_TAG"
				}
			}
		}
	}
}

