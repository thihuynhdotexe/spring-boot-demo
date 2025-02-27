pipeline {
  agent any

  tools {
    jdk 'jdk-11'
    maven 'mvn-3.6.3'
  }


  stages {
    stage('Unit Tests') {
      steps {
        sh 'mvn --version'
        sh "mvn test"
      }
    }
  
    stage('Build') {
      steps {
        withMaven(maven : 'mvn-3.6.3') {
          sh "mvn clean package -Dmaven.test.skip=true"
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
          withMaven(maven : 'mvn-3.6.3') {
            withCredentials([string(credentialsId: 'sonarqube-credentials', variable: 'SONAR_TOKEN')]) {
              sh "mvn sonar:sonar -Dsonar.host.url=http://sonar:9000 -Dsonar.login=$SONAR_TOKEN"
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
        writeFile file: 'anchore_images', text: 'docker.io/thihuynh/spring-boot-demo'
        anchore bailOnFail: false, bailOnPluginFail: false,name: 'anchore_images'
      }
    }

    stage('Deploy to K8s') {
      steps {
        kubernetesDeploy configs: 'k8s.yaml',  kubeconfigId: 'kubernetes-config'
      } 
    }
  }
}
