pipeline {
  agent any
  
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    
    stage('Build') {
      steps {
        // Build your project here
      }
    }
    
    stage('SonarQube Scan') {
      when {
        branch '.*' // Match all branches
      }
      steps {
        script {
          def scannerHome = tool 'SonarQubeScanner'
          
          withSonarQubeEnv('SonarQube Server') {
            sh "${scannerHome}/bin/sonar-scanner"
          }
        }
      }
    }
    
    stage('Deploy') {
      steps {
        // Deploy your project here
      }
    }
  }
  
  post {
    always {
      // Publish SonarQube Quality Gate results
      script {
        withSonarQubeEnv('SonarQube Server') {
          def qg = waitForQualityGate()
          if (qg.status != 'OK') {
            error "Pipeline aborted due to quality gate failure: ${qg.status}"
          }
        }
      }
    }
  }
}

def maven_settings_xml = readFile "${env.JENKINS_HOME}/maven-settings.xml"


withSonarQubeEnv('my-sonarqube-server') {
    sh "mvn sonar:sonar -s $maven_settings_xml"
}

