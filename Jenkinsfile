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
    stage('Deploy to EC2') {
        steps {
            sh 'aws ec2 run-instances --image-id ami-0c55b159cbfafe1f0 --count 1 --instance-type t2.micro --key-name MyKeyPair --security-group-ids sg-0123456789abcdef0 --subnet-id subnet-0123456789abcdef0'
            sh 'aws ec2 describe-instances --query "Reservations[*].Instances[*].PublicIpAddress" --output=text > ec2_instance_ip.txt'
            sh 'DOCKER_HOST_IP=$(cat ec2_instance_ip.txt)'
            sh 'docker -H tcp://$DOCKER_HOST_IP:2375 run -d -p 8080:8080 my-java-app'
        }
    }
}
