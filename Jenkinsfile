pipeline {
    any {
        // We use a Docker agent with Maven and Google Cloud SDK pre-installed.
        // This ensures a clean and consistent build environment.
        docker {
            image 'gcr.io/cloud-builders/mvn:latest'
            args '--volume /var/jenkins_home/workspace/your-job-name/.m2:/root/.m2' // Cache Maven dependencies
        }
    }

    environment {
        // Use Jenkins Credentials to securely store the GCP service account key
        GCP_CREDENTIALS = credentials('gcp-service-account-key') 
        GCP_PROJECT = 'your-project-id'
        ARTIFACT_REGISTRY_REPO = 'your-repo-name'
        ARTIFACT_REGISTRY_LOCATION = 'us-central1'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                // The pipeline automatically checks out the code based on the job configuration.
                // We're keeping it explicit here for clarity.
                checkout scm
            }
        }
        
        stage('Build and Test') {
            steps {
                echo 'Building with Maven and running tests...'
                // The -B flag makes Maven run in non-interactive batch mode.
                sh 'mvn clean install -B'
            }
        }

        stage('Push Artifact to GCP Artifact Registry') {
            steps {
                echo 'Configuring Docker for GCP Artifact Registry...'
                // Authenticate Docker with the Artifact Registry.
                // The sh step assumes the gcloud CLI is available in the agent's environment.
                withCredentials([file(credentialsId: env.GCP_CREDENTIALS, variable: 'GCP_KEY_FILE')]) {
                    sh "gcloud auth activate-service-account --key-file=\$GCP_KEY_FILE"
                    sh "gcloud auth configure-docker ${env.ARTIFACT_REGISTRY_LOCATION}-docker.pkg.dev"
                }

                echo 'Deploying artifact to Artifact Registry...'
                // Maven's deploy goal uses the distributionManagement configuration from pom.xml.
                sh 'mvn deploy'
            }
        }
    }

    post {
        always {
            echo "Pipeline finished for ${env.JOB_NAME} build #${env.BUILD_NUMBER}"
        }
        success {
            echo "Build and deployment to Artifact Registry was successful! 🎉"
        }
        failure {
            echo "Build failed. Check the console output for details. ❌"
        }
    }
}
