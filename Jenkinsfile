pipeline {
  agent any
  tools {
    maven 'maven'
  }

  stages {
    stage('Hello') {
      steps {
        sh 'mvn clean package'
      }
    }
  }
}
