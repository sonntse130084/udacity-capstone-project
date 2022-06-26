pipeline {
	agent any
	environment {
       KUBECONFIG='~/.kube/config'
       THE_BUTLER_SAYS_SO=credentials('udacity-capstone')//can be used in whole pipeline
   }
	stages {

		stage('Lint HTML') {
			steps {
				sh 'tidy -q -e ./docker/blue/*.html'
			}
		}

	stage('build image'){
		parallel {
			stage('Build Docker Image blue') {
				steps {
					withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'udacity-docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
						sh '''
							docker build -t nguyenson99/udacity-capstone-docker-blue ./docker/blue/
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
        }

	}

		
	stage('push image'){
        parallel {
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
        }
	}

		stage('Set Current kubectl Context') {
			steps {
                    withEnv(["KUBECONFIG=$HOME/.kube/kubeconfig"]) {
                    // Your stuff here
                    sh '''
						kubectl config use-context arn:aws:eks:us-east-1:900569321428:cluster/udacitycluster
					'''
                    }	

			}
		}

		stage('Deploy Blue Container') {
			steps {

                    withEnv(["KUBECONFIG=$HOME/.kube/kubeconfig"]) {
                    // Your stuff here
                    sh '''
						kubectl apply -f ./kubernetes-resources/blue-replication-controller.yml 
					'''
                    }	

			}
		}

		stage('Deploy green container') {
			steps {

                    withEnv(["KUBECONFIG=$HOME/.kube/kubeconfig"]) { 
					sh '''
						kubectl apply -f ./kubernetes-resources/green-replication-controller.yml 
					'''
				    }

			}
		}

		stage('Create Service Pointing to Blue Replication Controller') {
			steps {

                    withEnv(["KUBECONFIG=$HOME/.kube/kubeconfig"]) {
					sh '''
						kubectl apply -f ./kubernetes-resources/blue-service.yml 
					'''
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

					withEnv(["KUBECONFIG=$HOME/.kube/kubeconfig"]) {
                    sh '''
						kubectl apply -f ./kubernetes-resources/green-service.yml 
					'''
                    }
                }

		}

	}
}
