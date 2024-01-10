pipeline {
    agent any
	
    parameters {
        // Define parameters that allow users to input the Docker image name and Git repository URL
        string(name: 'DOCKER_IMAGE', description: 'The name of the Docker image to build')
        string(name: 'GIT_REPO', description: 'The Git repository URL containing the Dockerfile')
		string(name: 'PROJECT_NAME', description: 'Target project name in Docker registry')
		string(name: 'DOCKER_TAG', description: 'The tags of the Docker image to build')
    }

    stages {
	
		stage('Pre-flight Check') {
            steps {
                script {
                    // Check if the DOCKER_IMAGE parameter is provided
                    if (!params.DOCKER_IMAGE) {
                        error "The DOCKER_IMAGE parameter is missing. Please provide the name of the Docker image to build."
                    }
                    // Check if the GIT_REPO parameter is provided
                    if (!params.GIT_REPO) {
                        error "The GIT_REPO parameter is missing. Please provide the Git repository URL containing the Dockerfile."
                    }
					
					if (!params.PROJECT_NAME) {
                        error "The PROJECT_NAME parameter is missing. Please provide the target project name for your built image."
                    }
					
					if (!params.DOCKER_TAG) {
                        error "The DOCKER_TAG parameter is missing. Please provide the target project name for your built image."
                    }
                }
            }
        }
		
        stage('Checkout code') {
            steps {
                // Checkout the code from the specified Git repository and branch
                git url: env.GIT_REPO
            }
        }
        
        stage('Build image') {
            steps {
                script {
                    // Build the Docker image from the Dockerfile in the repository
                    docker.build("${env.DOCKER_REGISTRY}/${env.PROJECT_NAME}/${env.DOCKER_IMAGE}:${env.DOCKER_TAG}")
                }
            }
        }

        stage('Push image') {
            steps {
                script {
                    // Log in to Docker registry
                    docker.withRegistry("https://${env.DOCKER_REGISTRY}", env.DOCKER_REGISTRY_CREDENTIALS_ID) {
                        // Tag the image with the build number and latest
                        def dockerImage = docker.image("${env.DOCKER_REGISTRY}/${env.PROJECT_NAME}/${env.DOCKER_IMAGE}:${env.DOCKER_TAG}")

                        dockerImage.push()
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up the Docker images from the Jenkins agent
            cleanWs()
        }
        success {
            echo 'Build and push succeeded!'
        }
        failure {
            echo 'Build or push failed.'
        }
    }
}