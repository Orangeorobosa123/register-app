pipeline {
    agent { label 'Jenkins-Agent' }
    tools {
        jdk 'Java11'
        maven 'Maven3'
    }

    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "ericxtx"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")// Replace with the actual API token
        
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/Orangeorobosa123/register-app.git'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'Jenkins_Lord') { 
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Scan Docker Image with Trivy") {
            steps {
                script {
                    // Install Trivy without sudo
                    sh 'curl -s https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh'

                    // Run Trivy scan on the Docker image
                    sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${IMAGE_NAME}:${IMAGE_TAG} > trivy_scan_report.txt || true"
                    
                    // Archive the scan results
                    archiveArtifacts artifacts: 'trivy_scan_report.txt', allowEmptyArchive: true
                    
                    // Print the Trivy scan result to console for real-time output
                    sh 'cat trivy_scan_report.txt'
                }
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image = docker.build("${IMAGE_NAME}")
                    }
                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }

        stage("Clean Artifacts") {
            steps {
                script {
                    // Clean up any temporary build or Docker images to free up space
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"  // Removes the built image
                    sh "docker rmi ${IMAGE_NAME}:latest || true"         // Removes the latest image
                    sh "mvn clean"                                       // Cleans up Maven build artifacts
                    cleanWs()                                           // Clean up Jenkins workspace
                }
            }
        }

        stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh """
                    curl -v -k --user admin:${JENKINS_API_TOKEN} \
                    -X POST -H 'cache-control: no-cache' \
                    -H 'content-type: application/x-www-form-urlencoded' \
                    --data 'IMAGE_TAG=${IMAGE_TAG}' \
                    'http://ec2-3-94-196-43.compute-1.amazonaws.com:8080/job/gitops-register-app-cd/buildWithParameters?token=gitops-token'
                    """
                }
            }
        }
    }
}
