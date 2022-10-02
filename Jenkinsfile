pipeline {
  agent {
    kubernetes {
      yamlFile 'pods.yaml'
    }      
  }
  stages{
    stage('Checkout from SCM'){
      when{
        expression{
          false
        }
      }
      steps{
        container('git'){
           git branch: 'main', url: 'https://github.com/ullagallu123/spring-petclinic.git'
        }  
      }
    }
    stage('Build Maven'){
      when{
        expression{
          false
        }
      }
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
            sh 'which sonar-scanner'
        }
    }
  }
}