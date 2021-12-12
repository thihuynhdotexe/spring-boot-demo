pipeline {
  agent any

  tools {
    jdk 'jdk-11'
    maven 'mvn-3.6.3'
  }


  stages {
    // stage('Unit Tests') {
    //   steps {

    //     withEnv(['JAVA_HOME=${ tool "jdk-11" }', 'PATH+MAVEN=${ tool "mvn-3.6.3" }/bin:${env.JAVA_HOME}/bin']) {
    //        sh "mvn test"
    //     }
    //   }
    // }

    stage('Unit Tests 2') {
      steps {
        sh 'echo $JAVA_HOME'
        sh 'mvn --version'
        sh "mvn test"
      }
    }
  
    stage('Build') {
      steps {
        withMaven(maven : 'mvn-3.6.3') {
          sh "mvn package"
        }
      }
    }

    stage ('OWASP Dependency-Check Vulnerabilities') {
      steps {
        withMaven(maven : 'mvn-3.6.3') {
          sh 'mvn dependency-check:check'
        }

        dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
      }
    }

    stage('SonarQube analysis') {
      steps {
        withSonarQubeEnv(credentialsId: 'sonarqube-secret', installationName: 'sonarqube-server') {
          withMaven(maven : 'mvn-3.6.3') {
            sh 'mvn sonar:sonar -Dsonar.dependencyCheck.jsonReportPath=target/dependency-check-report.json -Dsonar.dependencyCheck.xmlReportPath=target/dependency-check-report.xml -Dsonar.dependencyCheck.htmlReportPath=target/dependency-check-report.html'
          }
        }
      }
    }

    stage('Create and push container') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
          withMaven(maven : 'mvn-3.6.3') {
            sh "mvn jib:build"
          }
        }
      } 
    }

    stage('Anchore analyse') {
      steps {
        writeFile file: 'anchore_images', text: 'docker.io/maartensmeets/spring-boot-demo'
        anchore name: 'anchore_images'
      }
    }

    stage('Deploy to K8s') {
      steps {
        withKubeConfig([credentialsId: 'kubernetes-config']) {
          sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"'
          sh 'chmod u+x ./kubectl'
          sh './kubectl apply -f k8s.yaml'
        }
      } 
    }
  }
}
