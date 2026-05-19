pipeline {

    agent any

    environment {

        AWS_REGION = "ap-south-1"

        IMAGE_NAME = "helloapp"

        ECR_REGISTRY = "583067668082.dkr.ecr.ap-south-1.amazonaws.com"

        ECR_REPO = "583067668082.dkr.ecr.ap-south-1.amazonaws.com/helloapp"

        DEPLOY_SERVER = "ubuntu@13.203.196.159"

        SONAR_SCANNER = tool 'sonar-scanner'
    }

    stages {

        stage('Checkout') {

            steps {

                git branch: 'main',

                url: 'https://github.com/vnataraj5-ship-it/ECR-.git'
            }
        }

        stage('SonarQube Analysis') {

            steps {

                withSonarQubeEnv('sonar-server') {

                    sh """
                    ${SONAR_SCANNER}/bin/sonar-scanner \
                    -Dsonar.projectKey=helloapp \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://65.0.30.181:9000
                    """
                }
            }
        }

        stage('Docker Build') {

            steps {

                sh """
                docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                """
            }
        }

        stage('Push Image to ECR') {

            steps {

                sh """

                aws ecr get-login-password \
                --region ${AWS_REGION} |

                docker login \
                --username AWS \
                --password-stdin \
                ${ECR_REGISTRY}

                docker tag \
                ${IMAGE_NAME}:${BUILD_NUMBER} \
                ${ECR_REPO}:${BUILD_NUMBER}

                docker push \
                ${ECR_REPO}:${BUILD_NUMBER}

                """
            }
        }

        stage('Deploy to EC2') {

            steps {

                sshagent(['deploy-server']) {

                    sh """

                    ssh -o StrictHostKeyChecking=no \
                    ${DEPLOY_SERVER} "

                    aws ecr get-login-password \
                    --region ${AWS_REGION} |

                    docker login \
                    --username AWS \
                    --password-stdin \
                    ${ECR_REGISTRY}

                    docker stop helloapp || true

                    docker rm helloapp || true

                    docker pull \
                    ${ECR_REPO}:${BUILD_NUMBER}

                    docker run -d \
                    --name helloapp \
                    -p 5000:5000 \
                    ${ECR_REPO}:${BUILD_NUMBER}

                    "

                    """
                }
            }
        }
    }
}
