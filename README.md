
To deploy Nginx using Docker on a Jenkins pipeline,

### Prerequisites

1. **Jenkins**: Installed and configured with Docker installed on the Jenkins server or agent.
2. **Docker**: Docker should be installed on the Jenkins node running this pipeline.
3. **Jenkins Docker Permissions**: Ensure the Jenkins user has permissions to run Docker commands (e.g., added to the `docker` group).

 sudo chown jenkins /var/run/docker.sock
---

### Step 1: Create a Jenkins Pipeline Job

1. Go to Jenkins and create a **New Item**.
2. Choose **Pipeline** and name the job.
3. Scroll down to the **Pipeline** section and select **Pipeline script**.

---

### Step 2: Define the Pipeline Script

Here’s a basic Jenkins pipeline script to pull and run an Nginx container using Docker.

```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "nginx:latest"  // Set the Nginx Docker image version here
        CONTAINER_NAME = "nginx_server"  // Name for the Nginx container
        NGINX_PORT = "8080"  // Port on the host to map to Nginx (default is 80)
    }

    stages {
        stage('Pull Nginx Docker Image') {
            steps {
                script {
                    echo "Pulling the Nginx Docker image..."
                    sh "docker pull $DOCKER_IMAGE"
                }
            }
        }

        stage('Run Nginx Container') {
            steps {
                script {
                    echo "Running the Nginx Docker container..."
                    
                    // Check if a container with the same name is already running and stop/remove it
                    sh """
                        if [ \$(docker ps -aq -f name=$CONTAINER_NAME) ]; then
                            docker stop $CONTAINER_NAME || true
                            docker rm $CONTAINER_NAME || true
                        fi
                    """

                    // Run the Nginx container
                    sh "docker run -d --name $CONTAINER_NAME -p $NGINX_PORT:80 $DOCKER_IMAGE"
                }
            }
        }

        stage('Verify Nginx Container is Running') {
            steps {
                script {
                    echo "Verifying that Nginx is running on port $NGINX_PORT..."
                    sh "docker ps -f name=$CONTAINER_NAME"
                }
            }
        }
    }

    post {
        success {
            echo "Nginx deployment completed successfully."
        }
        failure {
            echo "Nginx deployment failed."
        }
        always {
            echo "Pipeline execution completed."
        }
    }
}
```

---
