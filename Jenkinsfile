pipeline {
    agent any
    tools {
        jdk 'Java11'
        maven 'Maven3'
    }

    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "ericxtx"  // Your DockerHub username
        DOCKER_PASS = 'dockerhub'  // Your DockerHub password or access token
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
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

        stage("Build Docker Image") {
            steps {
                script {
                    docker_image = docker.build("${IMAGE_NAME}")
                }
            }
        }

        stage("Scan Docker Image with Trivy") {
            steps {
                script {
                    // Install Trivy if not already installed
                    sh 'curl -s https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sudo bash'

                    // Run Trivy scan
                    sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${IMAGE_NAME}:${IMAGE_TAG}"

                    // If Trivy finds critical or high vulnerabilities, the build will fail due to exit-code 1.
                }
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    // Push the image only if the scan passed
                    docker.withRegistry('', DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }
    }
}
