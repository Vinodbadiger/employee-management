pipeline {
    agent any

    tools {
        jdk 'JDK21'
        maven 'Maven3'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Vinodbadiger/employee-management.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Verify Artifact') {
            steps {
                sh '''
                    echo "Checking target directory..."
                    ls -lh target

                    echo "Checking for JAR..."
                    test -f target/*.jar
                '''
            }
        }
        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t employee-management:${BUILD_NUMBER} .
                   '''
                  }  
            }
    }
}
