pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'roseaw-dockerhub'
        DOCKER_IMAGE = 'cithit/minterjt'                               //<-----change this to your MiamiID!
        IMAGE_TAG = "lab34-build-${BUILD_NUMBER}"
        GITHUB_URL = 'https://github.com/srxxm49mps-png/225-lab3-4.git' //<-----change this to match this new repository!
        KUBECONFIG = credentials('minterjt')                           //<-----change this to match your kubernetes credentials (MiamiID-225)!  1 More change on line 63!
    }

    parameters {
        string(name: 'PRODUCTION_IP', defaultValue: '', description: 'Course-assigned MetalLB IP; required for production deployment')
        booleanParam(name: 'VERIFY_DEV_ONLY', defaultValue: true, description: 'Verify DEV while production IP assignment is unresolved')
    }

    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: "${GITHUB_URL}"]]])
            }
        }

        stage('Lint HTML') {
            steps {
                sh 'npm install htmlhint --save-dev'
                sh 'npx htmlhint *.html'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to Dev Environment') {
            steps {
                script {
                    // This sets up the Kubernetes configuration using the specified KUBECONFIG
                    // This updates the deployment-dev.yaml to use the new image tag
                    sh "sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|' deployment-dev.yaml"
                    sh "kubectl apply -f deployment-dev.yaml"
                    sh "kubectl rollout status deployment/minterjt-lab34-dev --timeout=180s"
                }
            }
        }
        stage('Deploy to Prod Environment') {
            when { expression { !params.VERIFY_DEV_ONLY } }
            steps {
                script {
                    // Set up Kubernetes configuration using the specified KUBECONFIG
                    //sh "ls -la"
                    if (!params.PRODUCTION_IP?.trim()) {
                        error('Production requires the course-assigned MetalLB IP. DEV verification is available with VERIFY_DEV_ONLY.')
                    }
                    if (!(params.PRODUCTION_IP ==~ /(?:[0-9]{1,3}\.){3}[0-9]{1,3}/)) {
                        error('PRODUCTION_IP must be an IPv4 address.')
                    }
                    sh "kubectl get ipaddresspools.metallb.io -A"
                    sh "sed -i 's|PRODUCTION_IP|${params.PRODUCTION_IP}|' deployment-prod.yaml"
                    sh "sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|' deployment-prod.yaml"
                    sh "kubectl apply -f deployment-prod.yaml"
                    sh "kubectl rollout status deployment/minterjt-lab34-prod --timeout=180s"
                }
            }
        }
        stage('Check Kubernetes Cluster') {
            steps {
                script {
                    sh "kubectl get pods"
                    sh "kubectl get services"
                    sh "kubectl get deploy"
                }
            }
        }
    }

    post {

        success {
            slackSend color: "good", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER} (DEV-only: ${params.VERIFY_DEV_ONLY})"
        }

        unstable {
            slackSend color: "warning", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER} (DEV-only: ${params.VERIFY_DEV_ONLY})"
        }

        failure {
            slackSend color: "danger", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER} (DEV-only: ${params.VERIFY_DEV_ONLY})"
        }
    }
}
