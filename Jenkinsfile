pipeline{
    
    agent any 

    parameters{

        choice(name: 'action', choices: 'create\ndestroy\ndestroyekscluster', description: 'create/update or destroy the eks cluster')
        string(name: 'cluster', defaultValue: 'demo-cluster', description: 'Eks cluster name')
        string(name: 'region', defaultValue: 'us-east-1', description: 'Eks cluster region')
    }
    environment{

        ACCESS_KEY = credentials('aws_access_key_id')
        SECRET_KEY = credentials('aws_secret_access_key')
    }
    stages {
        
        stage('Git Checkout'){
            
            steps{
                
                script{
                    
                    git branch: 'main', url: 'https://github.com/shkr04041996/Cicd_jenkins.git'
                }
            }
        }
        stage('UNIT testing'){
            
            steps{
                
                script{
                    
                    sh 'mvn test'
                }
            }
        }
        stage('Integration testing'){
            
            steps{
                
                script{
                    
                    sh 'mvn verify -DskipUnitTests'
                }
            }
        }
        stage('Maven build'){
            
            steps{
                
                script{
                    
                    sh 'mvn clean install'
                }
            }
        }
        stage('Static code analysis'){
            
            steps{
                
                script{
                    
                    withSonarQubeEnv(credentialsId: 'sonar-api') {
                        
                        sh 'mvn clean package sonar:sonar'
                    }
                   }
                    
                }
            }
            stage('Quality Gate Status'){
                
                steps{
                    
                    script{
                        
                        waitForQualityGate abortPipeline: false, credentialsId: 'sonar-api'
                    }
                }
            }
            stage('docker image build'){

                steps{

                    script{

                        sh 'docker image build -t $JOB_NAME:v1.$BUILD_ID .'
                        sh 'docker image tag $JOB_NAME:v1.$BUILD_ID shkr04041996/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker image tag $JOB_NAME:v1.$BUILD_ID shkr04041996/$JOB_NAME:latest'
                    }

                }

            }
            stage('push image to the dockerhub'){

                steps{

                    script{

                        withCredentials([string(credentialsId: 'dockerhub_cred', variable: 'docker_hub_cred')]) {

                            sh 'docker login -u shkr04041996 -p ${docker_hub_cred}'
                            sh 'docker image push shkr04041996/$JOB_NAME:v1.$BUILD_ID'
                            sh 'docker image push shkr04041996/$JOB_NAME:latest'

                    }
                    }
                }
            }
            stage('eks connect'){
                steps{
                   sh """
                       aws configure set aws_access_key_id "$ACCESS_KEY"
                       aws configure set aws_secret_access_key "$SECRET_KEY"
                       aws configure set region ""
                       aws eks --region ${params.region} update-kubeconfig --name ${params.cluster}
                      """
                }
            }
//            stage('eks deployments'){
//              when{ expression { params.action == 'create'}}
//                steps{

//                    script{
//                        def apply = false
//                        try{
//                           input message: 'please confirm the apply to initiate the deployments', ok: 'Ready to apply the config'
//                           apply = true 
//                        }
//                        catch(err){
//                           apply = false
//                           CurrentBuild.result='UNSTABLE'
//                       }
//                        if(apply){

//                            sh """
//                                   cd /var/lib/jenkins/workspace/demo_project
//                                   pwd
//                                   Kubectl apply -f  .
//                                """
//                        }
//                    }
//                }
//        }
    }
}
        
