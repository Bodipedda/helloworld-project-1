pipeline {
  agent any

  tools {
    jdk 'Java'
    maven 'Maven'
  }
  
  environment {

      sonar_url = 'http://10.128.15.219:9000'
      sonar_username = 'admin'
      sonar_password = 'admin'
      nexus_url = '10.128.15.219:8081'
      artifact_version = '4.0.0'

 }
 parameters {
      string(defaultValue: 'main', description: 'Please type any branch name to deploy', name: 'Branch')
 }  

stages {
    stage('Git checkout'){
      steps {
        git branch: '${Branch}',
        url: 'https://github.com/chinni4321/helloworld.git'
      }
    }
    stage('Maven build'){
      steps {
        sh 'mvn clean install'
      }
    }
  stage ('Sonarqube Analysis'){
           steps {
           withSonarQubeEnv('Sonarqube') {
           sh '''
           mvn -e -B sonar:sonar -Dsonar.java.source=1.8 -Dsonar.host.url="${sonar_url}" -Dsonar.login="${sonar_username}" -Dsonar.password="${sonar_password}" -Dsonar.sourceEncoding=UTF-8
           '''
           }
         }
      } 
      stage ('Publish Artifact') {
        steps {
          nexusArtifactUploader artifacts: [[artifactId: 'hello-world-war', classifier: '', file: "target/hello-world-war-1.0.0.war", type: 'war']], credentialsId: 'nexus-cred', groupId: 'com.efsavage', nexusUrl: "${nexus_url}", nexusVersion: 'nexus3', protocol: 'http', repository: 'release', version: "${artifact_version}"
        }
      }
      
      stage ('Build Docker Image'){
        steps {
          sh '''
          cd ${WORKSPACE}
          docker build -t gcr.io/ferrous-syntax-348017/helloworld --file=Dockerfile ${WORKSPACE}
          '''
        }
      }
      stage ('Publish Docker Image'){
        steps {
          sh '''
          docker push gcr.io/ferrous-syntax-348017/helloworld
          '''
        }
      }
      stage ('Deploy to kubernetes'){
        steps{
          script {
            sh "kubectl config use-context gke_ferrous-syntax-348017_us-central1-c_demo14"
            sh "cd ${WORKSPACE}"
            sh "kubectl delete -f '${WORKSPACE}'/hello.yaml"
            sh "kubectl apply -f '${WORKSPACE}'/hello.yaml"
          }
         }
        }
     }
   }
Footer
