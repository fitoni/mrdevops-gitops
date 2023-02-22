pipeline {
    agent any
    environment{
        VERSION = "${env.BUILD_NUMBER}"
    }
    tools{
        maven 'maven'
    }
    
    stages{
        stage('Build Maven'){
            steps{
                script{
                    sh 'mvn clean install'
                }
            }
        }

        stage('Build Docker Image'){
            steps{
                script{
                    sh 'docker build -t fitoni/sureshrajuvetukuri:${VERSION} .'                    
                }
            }
        }

         stage('Push Docker Image onto Hub'){
            steps{
                script{
                     withCredentials([string(credentialsId: 'dockerhubpassword', variable: 'dockerhubpassword')]) {
                        sh '''
                            echo $dockerhubpassword | docker login -u fitoni --password-stdin https://index.docker.io/v1/
                            docker push fitoni/sureshrajuvetukuri:${VERSION}
                            docker rmi fitoni/sureshrajuvetukuri:${VERSION}
                        '''                        
                    } 
                }
            }
        }

        stage('Deploy to GCP-Kubernetes Cluster'){
            steps{
                script{
                    kubernetesDeploy (configs: 'deploymentservice.yaml', kubeconfigId: 'kubeconfig')
                }
            }
        }  
    }
}
