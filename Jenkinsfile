pipeline {
  agent {
    kubernetes {
      yamlFile 'pods.yaml'
    }      
  }
  environment{
    NEXUS_VERSION = "nexus3"
    NEXUS_PROTOCOL = "http"
    NEXUS_URL = "139.59.108.217:32363"
    NEXUS_REPOSITORY= "maven-hosted"
    NEXUS_CREDENTIAL_ID = "nexus-cred"
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
    stage('publish artifact to nexus'){
      steps{
        container('jnlp'){
          script {
             pom = readMavenPom file: "pom.xml";
             filesByGlob = findFiles(glob: "target/*.${pom.packaging}"); 
             echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
             artifactPath = filesByGlob[0].path;
             artifactExists = fileExists artifactPath;
             if(artifactExists) {   
                echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                nexusArtifactUploader(
                  nexusVersion: NEXUS_VERSION,
                  protocol: NEXUS_PROTOCOL,
                  nexusUrl: NEXUS_URL,
                  groupId: pom.groupId,
                  version: pom.version,
                  repository: NEXUS_REPOSITORY,
                  credentialsId: NEXUS_CREDENTIAL_ID,
                  artifacts: [
                   [artifactId: pom.artifactId,
                   classifier: '',
                   file: artifactPath,
                   type: pom.packaging],

                   [artifactId: pom.artifactId,
                   classifier: '',
                   file: "pom.xml",
                   type: "pom"]
                  ]
                );
              } 
              else {
                 error "*** File: ${artifactPath}, could not be found";
              }
          }
        }
      }
    }
  }
}