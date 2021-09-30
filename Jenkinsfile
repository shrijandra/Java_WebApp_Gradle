currentBuild.displayName = "Spring_gradle # "+currentBuild.number
        
pipeline{
        agent any  
        environment { 
            VERSION = "${env.BUILD_ID}"
            }
        
        stages{
	
	
              stage('Quality Gate Statuc Check'){

               agent {
                  docker {
                  image 'openjdk:11'
	          args '-v $HOME/.m2:/root/.m2'
                }
            }
                steps{
                  script{
                    withSonarQubeEnv('sonarserver') { 
                      sh "chmod +x gradlew"
                      sh "java -version"
                      sh "./gradlew sonarqube"
                       }
                      timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                  }
                }  
              }
		    stage('docker image creation stage'){
                steps{
                    script{
                        withCredentials([string(credentialsId: 'docker_password', variable: 'docker_password')]) {
			
                        sh '''
                        docker build -t 34.125.132.246:8083/springapp:${VERSION} .
                        docker login -u admin -p $docker_password 34.125.132.246:8083
                        docker push 34.125.132.246:8083/springapp:${VERSION}
			docker rmi 34.125.132.246:8083/springapp:${VERSION}
                        ''' 
                        }
                    }
                }

            }

        stage('checking misconfigurations of k8s manifest using datree'){
          steps{
            script{
              dir ("kubernetes/"){
                sh 'helm datree test myapp'
              }
            }
          }
        }
		
      stage('pushing helm charts to artifactory'){
	steps{
	  script{
            withCredentials([string(credentialsId: 'docker_password', variable: 'docker_password')]) {
              dir ("kubernetes/"){
		  sh '''
		  helmversion=$(helm show chart myapp | grep version | cut -d: -f2 | tr -d ' ')
		  tar -czvf myapp-${helmversion}.tgz myapp/
		  curl -u admin:$docker_password http://34.125.132.246:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
		  '''
	        }
	      }
            }
	  }
        }
		
		stage('build') {
			steps {
			    script {
			      timeout(time: 10, unit: 'MINUTES') {
			        mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL} By going jenkins URL please approve the deployment", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "deekshith.snsep@foomail.com";
				input(id: "Deploy Gate", message: "Deploy ${params.project_name}?", ok: 'Deploy')
			      }
			    }
			}
		    }

	
		stage('connecting to k8s cluster'){
		  steps{
	            script{
	    		withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
			  dir ("kubernetes/"){  
				sh 'helm list'
				sh 'helm upgrade --install --set image.repository="34.125.132.246:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/ ' 
			  }
		       } 
		    }		
		  }
		}
		
		stage('Testing application health'){
	    	  steps{
	 	    script{
		      withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
			sh 'sleep 60'
			sh 'kubectl run curl --image=curlimages/curl -i --rm --restart=Never -- curl myjavaapp-myapp:8080'
		      }
		    }	  
		  }
		} 
	  	
      }
	
	
	post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "deekshith.snsep@foomail.com";  
		}
	}
	
    }
