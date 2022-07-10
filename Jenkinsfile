pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
            agent {
                docker {
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                withSonarQubeEnv(credentialsId: 'sonar-token') {
                         sh 'chmod +x gradlew'
                         sh './gradlew sonarqube'
                    }     
                }
            }
        }
        stage("docker build & docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                             sh '''
                               docker build -t 34.71.200.141:8083/springapp:${VERSION} .
                               docker login -u admin -p $docker_password 34.71.200.141:8083
                               docker push 34.71.200.141:8083/springapp:${VERSION}
                               docker rmi 34.71.200.141:8083/springapp:${VERSION}
                            ''' 
                    }
                  

                }
            }
        }
        stage('identifying misconfig using datree in helm charts'){
            steps{
                script{
                    dir('kubernetes/') {
                        withEnv(['datree token=0fb7a18c-2fb9-44e8-888b-267051b00018']) {
                            sh 'helm datree test myapp/'
                        }
                    }
                }
            }           
        }



    }

}
