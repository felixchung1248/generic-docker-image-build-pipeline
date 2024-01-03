pipeline {
    agent any
	
    parameters {
        // Define parameters that allow users to input the Docker image name and Git repository URL
        string(name: 'DOCKER_IMAGE', defaultValue: 'my-image-name', description: 'The name of the Docker image to build')
        string(name: 'GIT_REPO', defaultValue: 'https://github.com/username/repo.git', description: 'The Git repository URL containing the Dockerfile')
    }
	
    environment {
        // Define variables for Docker image and registry
        // DOCKER_REGISTRY_CREDENTIALS_ID = 'docker_registry_cred' // The ID for Jenkins credentials to access your registry
        GIT_BRANCH = 'main' // Replace with your branch as needed
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
                }
            }
        }
		
        stage('Checkout code') {
            steps {
                // Checkout the code from the specified Git repository and branch
                git branch: env.GIT_BRANCH, url: env.GIT_REPO
            }
        }
        
        stage('Build image') {
            steps {
                script {
                    // Build the Docker image from the Dockerfile in the repository
                    docker.build("${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE}")
                }
            }
        }

        stage('Push image') {
            steps {
                script {
                    // Log in to Docker registry
                    docker.withRegistry(env.DOCKER_REGISTRY, env.DOCKER_REGISTRY_CREDENTIALS_ID) {
                        // Tag the image with the build number and latest
                        def dockerImage = docker.image("${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE}")
                        dockerImage.tag("latest")
                        dockerImage.tag("${env.BUILD_NUMBER}")
                        
                        // Push the tagged images to the registry
                        dockerImage.push("latest")
                        dockerImage.push("${env.BUILD_NUMBER}")
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