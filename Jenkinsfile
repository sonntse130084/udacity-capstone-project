pipeline {
	agent any
	environment {
       KUBECONFIG='~/.kube/kubeconfig'                               //can be used in whole pipeline
   }
	stages {

		stage('Lint HTML') {
			steps {
				sh 'tidy -q -e ./docker/blue/*.html'
			}
		}
		
		stage('Build Docker Image blue') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'udacity-docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker build -t nguyenson99/udaciz`ty-capstone-docker-blue ./docker/blue/
					'''
				}
			}
		}

		stage('Push Image To Dockerhub blue') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'udacity-docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						docker push nguyenson99/udacity-capstone-docker-blue
					'''
				}
			}
		}

		stage('Build Docker Image green') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'udacity-docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker build -t nguyenson99/udacity-capstone-docker-green ./docker/green/
					'''
				}
			}
		}

		stage('Push Image To Dockerhub green') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'udacity-docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						docker push nguyenson99/udacity-capstone-docker-green
					'''
				}
			}
		}

		stage('Set Current kubectl Context') {
			steps {
				withAWS(region:'us-east-1', credentials:'udacity-capstone') {
                    withEnv(["KUBECONFIG=$HOME/.kube/kubeconfig"]) {
                    // Your stuff here
                    sh '''
						kubectl config use-context arn:aws:eks:us-east-1:125745568001:cluster/udacitycluster
					'''
                    }	
				}
			}
		}

		stage('Deploy Blue Container') {
			steps {
				withAWS(region:'us-east-1', credentials:'udacity-capstone') {
                    withEnv(["KUBECONFIG=$HOME/.kube/kubeconfig"]) {
                    // Your stuff here
                    sh '''
						kubectl apply -f ./kubernetes-resources/blue-replication-controller.yml 
					'''
                    }	
				}
			}
		}

		stage('Deploy green container') {
			steps {
				withAWS(region:'us-east-1', credentials:'udacity-capstone') {
                    withEnv(["KUBECONFIG=$HOME/.kube/kubeconfig"]) { 
					sh '''
						kubectl apply -f ./kubernetes-resources/green-replication-controller.yml 
					'''
				    }
                }
			}
		}

		stage('Create Service Pointing to Blue Replication Controller') {
			steps {
				withAWS(region:'us-east-1', credentials:'udacity-capstone') {
                    withEnv(["KUBECONFIG=$HOME/.kube/kubeconfig"]) {
					sh '''
						kubectl apply -f ./kubernetes-resources/blue-service.yml 
					'''
                    }
                }
			}
		}

		stage('Approval for Redirection') {
            steps {
                input "Ready to redirect traffic to green replication controller?"
            }
        }

		stage('Create Service Pointing to Green Replication Controller') {
			steps {
				withAWS(region:'us-east-1', credentials:'udacity-capstone') {
					withEnv(["KUBECONFIG=$HOME/.kube/kubeconfig"]) {
                    sh '''
						kubectl apply -f ./kubernetes-resources/green-service.yml 
					'''
                    }
                }
			}
		}

	}
}
