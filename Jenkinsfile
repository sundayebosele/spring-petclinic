pipeline {
agent any
 environment {
    DOCKER_HUB_USERNAME = credentials('docker_hub_username')
    DOCKER_HUB_PASSWORD = credentials('docker_hub_password')
}

stages {
    stage('Build') {
        steps {
            sh 'mvn clean install'
        }
    }
    stage('SonarQube Analysis') {
        steps {
            withSonarQubeEnv('SonarQube') {
                sh 'mvn sonar:sonar'
            }
        }
    }
    stage('Archive Artifacts') {
        steps {
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
    }
    stage('Docker Build') {
        steps {
            sh 'docker build -t my-java-app .'
        }
    }
    stage('Docker Push') {
        steps {
            sh 'docker login -u "$DOCKER_HUB_USERNAME" -p "$DOCKER_HUB_PASSWORD"'
            sh 'docker push my-java-app'
        }
    }
    stage('Trivy Scan') {
        steps {
            sh 'trivy my-java-app'
        }
    }
}
}
