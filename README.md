# jenkins-sonar-docker-pipeline-script
# Building an Automated CI/CD Pipeline with Jenkins, SonarQube, Docker, and AWS

pipeline {
    agent 'any' 
    tools{
        maven 'maven-3.9'
    }
        stages{
            stage('checkout the code'){
                steps{
                     git branch: 'main', url: 'https://github.com/nikhildhakad55/docker-project.git'
                }
            }
                stage('build the code'){
                    steps{
                        sh 'mvn clean package'
                    }
                }
            stage('sonar quality check'){
                steps{
                    withSonarQubeEnv('sonarqube-7.6') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
            stage('buid docker image'){
                steps{
                    sh 'docker image build -t nikhildhakad/sonar-image:latest .' #put your image here
                }
            }
            stage('PUSH IMAGE'){
                steps {
                    withCredentials([string(credentialsId: 'hubpwd', variable: 'hub')]) {
                      sh "docker login -u nikhildhakad -p ${hub}"                   # put here your crendtials
                      }
                      sh 'docker push nikhildhakad/sonar-image:latest'
                }
            }
            stage('create container'){
                steps {
                    sshagent(['ssh-do']) {
                      sh 'ssh -o StrictHostKeyChecking=no ec2-user@13.201.87.194 docker run -p 8000:80 -d --name docker-container nikhildhakad/sonar-image:latest'
                       }
                     }
                }
            }
            
       } 


