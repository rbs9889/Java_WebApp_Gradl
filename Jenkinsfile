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
                        docker build -t 34.125.27.120:8083/springapp:${VERSION} .
                        docker login -u admin -p $docker_password 34.125.27.120:8083
                        docker push 34.125.27.120:8083/springapp:${VERSION}
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
		  curl -u admin:$docker_password http://34.125.27.120:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
		  '''
	        }
	      }
            }
	  }
        }
	  	
      }
    }
