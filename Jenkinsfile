pipeline {

    agent any

    environment {

        AWS_REGION = 'ap-south-1'

        ECR_REGISTRY = '583067668082.dkr.ecr.ap-south-1.amazonaws.com'

        ECR_REPO = '583067668082.dkr.ecr.ap-south-1.amazonaws.com/helloapp'

        IMAGE_TAG = "${BUILD_NUMBER}"

        DEPLOY_SERVER = 'ubuntu@13.203.196.159'

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

                    sh '''

                    ${tool 'sonar-scanner'}/bin/sonar-scanner

                    '''

                }

            }

        }

        stage('Docker Build') {

            steps {

                sh '''

                docker build -t myapp:${BUILD_NUMBER} .

                '''

            }

        }

        stage('Push ECR') {

            steps {

                sh '''

                aws ecr get-login-password \
                --region $AWS_REGION |

                docker login \
                --username AWS \
                --password-stdin \
                $ECR_REGISTRY

                docker tag myapp:${BUILD_NUMBER} \
                $ECR_REPO:${BUILD_NUMBER}

                docker push \
                $ECR_REPO:${BUILD_NUMBER}

                '''

            }

        }

        stage('Deploy EC2') {

            steps {

                sshagent(['deploy-server']) {

                    sh '''

                    ssh -o StrictHostKeyChecking=no $DEPLOY_SERVER "

                    aws ecr get-login-password \
                    --region $AWS_REGION |

                    docker login \
                    --username AWS \
                    --password-stdin \
                    $ECR_REGISTRY

                    docker stop myapp || true

                    docker rm myapp || true

                    docker pull \
                    $ECR_REPO:${BUILD_NUMBER}

                    docker run -d \
                    --name myapp \
                    -p 5000:5000 \
                    $ECR_REPO:${BUILD_NUMBER}

                    "

                    '''

                }

            }

        }

    }

}
