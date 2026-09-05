pipeline {
    agent any
    
    options {

        buildDiscarder(logRotator(
            numToKeepStr: '10',
            artifactNumToKeepStr: '5'
        ))
    }  
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
	stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                        -u "$DOCKER_USER" \
                        --password-stdin
                     '''
                 }
             }
         }
	stage('Tag Image') {
            steps {
                sh '''
                  docker tag employee-management:${BUILD_NUMBER} \
                  vinodb48/employee-management:${BUILD_NUMBER}
                   '''
                  }
           }
	stage('Push Image') {
 	    steps {
       	        sh '''
        	  docker push vinodb48/employee-management:${BUILD_NUMBER}
       		   '''
   		 }
	  }
    stage('Deploy to Kubernetes') {
        steps {
            withCredentials([file(
                credentialsId: 'k8s-kubeconfig',
                variable: 'KUBECONFIG'
            )]) {
                sh '''
                   echo "Applying Kubernetes manifests..."
                   kubectl apply -f k8s/

                   echo "Updating deployment image..."
                   kubectl set image deployment/employee-management \
                     employee-management=vinodb48/employee-management:${BUILD_NUMBER} \
                     -n employee-app

                   echo "Waiting for rollout..."
                   kubectl rollout status deployment/employee-management \
                     -n employee-app \
                     --timeout=120s

                   echo "Deployment completed."
                   kubectl get pods -n employee-app -o wide
                '''
                 }
            }
        }
    }
}
