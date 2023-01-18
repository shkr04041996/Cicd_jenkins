pipeline{
    
    agent any 
    
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
            stage('upload war file to nexus'){

                steps{

                    script{
                        def readMavenPomVersion = readMavenPom file: 'pom.xml'
                       /* def nexusRepo = readMavenPomVersion.version.endsWith{"SNAPSHOT"} ? "demoapp-snapshot" : "demoapp-release"*/

                        nexusArtifactUploader artifacts: 
                        [
                            [
                                artifactId: 'springboot',
                                classifier: '',
                                file: 'target/Uber.jar', 
                                type: 'jar'
                                ]
                        ], 
                        credentialsId: 'nexus-auth',
                        groupId: 'com.example',
                        nexusUrl: '54.144.73.99:8081', 
                        nexusVersion: 'nexus3', 
                        protocol: 'http', 
                        repository: 'demoapp-release', 
                        version: "${readMavenPomVersion.version}"
                        /*version: '2.0.0'*/
                    }
                }
            }
            stage('docker image build'){

                steps{

                    script{

                        sh 'docker image build -t $JOB_NAME:v1.$BUILD_ID'
                        sh 'docker image tag $JOB_NAME:v1.$BUILD_ID shkr04041996/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker image tag $JOB_NAME:v1.$BUILD_ID shkr04041996/$JOB_NAME:latest'
                    }

                }

            }
        }
        
}