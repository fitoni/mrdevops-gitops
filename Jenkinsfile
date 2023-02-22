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
                    sh 'docker build -t fitoni/mrdevops-gitops:${VERSION} .'                    
                }
            }
        }

        stage('Push Docker Image onto Hub'){
            steps{
                script{
                     withCredentials([string(credentialsId: 'dockerhubpassword', variable: 'dockerhubpassword')]) {
                        sh '''
                            echo $dockerhubpassword | docker login -u fitoni --password-stdin https://index.docker.io/v1/
                            docker push fitoni/mrdevops-gitops:${VERSION}
                            docker rmi fitoni/mrdevops-gitops:${VERSION}
                        '''                        
                    } 
                }
            }
        }

        stage('Update newest tag in kubernetes deployment file in jenkins local worskpace'){
            steps{
                script{
                    sh """
                        cat deploymentservice.yaml
                        sed -i 's+fitoni/mrdevops-gitops.*+fitoni/mrdevops-gitops:${VERSION}+g' deploymentservice.yaml
                        cat deploymentservice.yaml
                    """                       
                }
            }
        }

        stage('Push updated kubernetes deployment file onto GitHub'){
            steps{
                script{
                    sh """
                        git config user.email 'fitoni77@gmail.com'
                        git config user.name 'fitoni'
                        git add deploymentservice.yaml
                        git commit -m 'Done by Jenkins Job changemanifest: ${env.BUILD_NUMBER}'
                    """     
                    withCredentials([gitUsernamePassword(credentialsId: 'github-account', gitToolName: 'Default')]) {
                        echo "Sukses login ke GitHub via Jenkins API"
                        echo "..........."
                        sh "git push https://github.com/fitoni/mrdevops-gitops.git main"
                    }                                      
                }
            }
        }

       /*  stage('Deploy to GCP-Kubernetes Cluster'){
            steps{
                script{
                    kubernetesDeploy (configs: 'deploymentservice.yaml', kubeconfigId: 'kubeconfig')
                }
            }
        }   */
    }
}
