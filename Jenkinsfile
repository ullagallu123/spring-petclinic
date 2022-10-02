pipeline {
  agent {
    kubernetes {
      yamlFile 'pods.yaml'
    }      
  }
  stages{
    stage('Checkout from SCM'){
      steps{
        container('git'){
           git branch: 'main', url: 'https://github.com/ullagallu123/spring-petclinic.git'
        }  
      }
    }
    stage('Build Maven'){
      steps{
        container('maven'){
           sh 'mvn -Dmaven.test.failure.ignore=true clean package'
        }  
      }
      post{
        success{
          junit '**/target/surefire-reports/*.xml'
        }
      } 
    }
    stage('sonar-testing'){
      steps{
        container('sonarcli'){
          withSonarQubeEnv(credentialsId: 'siva', installationName: 'sonarserver') { 
            sh '''/opt/sonar-scanner/bin/sonar-scanner \
             -Dsonar.projectKey=petclinic \
             -Dsonar.projectName=petclinic \
             -Dsonar.projectVersion=1.0 \
             -Dsonar.sources=src/main \
             -Dsonar.tests=src/test \
             -Dsonar.java.binaries=target/classes  \
             -Dsonar.language=java \
             -Dsonar.sourceEncoding=UTF-8 \
             -Dsonar.java.libraries=target/classes
            '''
          }
        }
      }
    } 
  }
}