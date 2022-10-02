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
    DOCKERHUB_USERNAME = "siva9666"
    APP_NAME = "petclinic"
    IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
    IMAGE_TAG = "${BUILD_NUMBER}"
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
    stage('sonar-analysis'){
      when{
        expression{
          false
        }
      }
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
      when{
        expression{
          false
        }
      }
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
    stage("DockerImage-build"){
      when{
        expression{
          false
        }
      }
      steps{
        container('dockercli'){
          sh "docker build -t $IMAGE_NAME:$IMAGE_TAG ."
          sh "docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest"
          withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
            sh "docker login -u $USER -p $PASS"
            sh "docker push $IMAGE_NAME:$IMAGE_TAG"
            sh "docker push $IMAGE_NAME:latest"
          }
          sh "docker rmi $IMAGE_NAME:$IMAGE_TAG"
          sh "docker rmi $IMAGE_NAME:latest"
        }
      }
    }
    stage("Deploying to k8s-cluster"){
      steps{
        container('kubectl'){
          withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', serverUrl: '') {
             sh "kubectl apply -f deployment.yaml"
          }
        }
      }
    }
  }
}