pipeline {

```
/*
 * The pipeline will eventually run on Jenkins Agent (Instance 2).
 * We will configure the agent label after confirming the label
 * assigned to your Instance 2.
 */
agent any

parameters {

    choice(
        name: 'ENVIRONMENT',
        choices: ['DEV', 'TEST', 'PROD'],
        description: 'Select the deployment environment'
    )
}

environment {

    IMAGE_NAME = 'jenkins-cicd-webapp'
    CONTAINER_NAME = 'jenkins-cicd-webapp'

    DEV_PORT = '5000'
    TEST_PORT = '5001'
    PROD_PORT = '5002'
}

stages {

    stage('Checkout') {

        steps {

            echo 'Checking out source code from GitHub...'

            checkout scm
        }
    }

    stage('Build') {

        steps {

            echo "Building Docker image: ${IMAGE_NAME}"

            sh '''
                docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
            '''
        }
    }

    stage('Parallel Tests') {

        parallel {

            stage('HTML Test') {

                steps {

                    echo 'Running HTML validation test...'

                    sh '''
                        test -f index.html
                        grep -q "<html" index.html
                        grep -q "DevOps CI/CD Pipeline" index.html
                    '''
                }
            }

            stage('Docker Test') {

                steps {

                    echo 'Testing Docker image...'

                    sh '''
                        docker run --rm ${IMAGE_NAME}:${BUILD_NUMBER} nginx -t
                    '''
                }
            }
        }
    }

    stage('Create Artifact') {

        steps {

            echo 'Creating deployment artifact...'

            sh '''
                mkdir -p artifact

                cp index.html artifact/
                cp Dockerfile artifact/

                echo "Application: ${IMAGE_NAME}" > artifact/build-info.txt
                echo "Build Number: ${BUILD_NUMBER}" >> artifact/build-info.txt
                echo "Environment: ${ENVIRONMENT}" >> artifact/build-info.txt
            '''

            archiveArtifacts artifacts: 'artifact/**',
                             fingerprint: true
        }
    }

    stage('Deploy DEV') {

        when {

            expression {
                params.ENVIRONMENT == 'DEV'
            }
        }

        steps {

            echo 'Deploying application to DEV...'

            sh '''
                docker rm -f ${CONTAINER_NAME}-dev 2>/dev/null || true

                docker run -d \
                    --name ${CONTAINER_NAME}-dev \
                    -p ${DEV_PORT}:80 \
                    ${IMAGE_NAME}:${BUILD_NUMBER}
            '''
        }
    }

    stage('Deploy TEST') {

        when {

            expression {
                params.ENVIRONMENT == 'TEST'
            }
        }

        steps {

            echo 'Deploying application to TEST...'

            sh '''
                docker rm -f ${CONTAINER_NAME}-test 2>/dev/null || true

                docker run -d \
                    --name ${CONTAINER_NAME}-test \
                    -p ${TEST_PORT}:80 \
                    ${IMAGE_NAME}:${BUILD_NUMBER}
            '''
        }
    }

    stage('Deploy PROD') {

        when {

            expression {
                params.ENVIRONMENT == 'PROD'
            }
        }

        steps {

            echo 'Deploying application to PROD...'

            sh '''
                docker rm -f ${CONTAINER_NAME}-prod 2>/dev/null || true

                docker run -d \
                    --name ${CONTAINER_NAME}-prod \
                    -p ${PROD_PORT}:80 \
                    ${IMAGE_NAME}:${BUILD_NUMBER}
            '''
        }
    }
}

post {

    success {

        echo "Pipeline completed successfully for ${params.ENVIRONMENT}"

        /*
         * Email notification will be enabled after we configure
         * Jenkins SMTP/email settings.
         */
    }

    failure {

        echo "Pipeline FAILED for ${params.ENVIRONMENT}"

        /*
         * Email notification will be enabled after we configure
         * Jenkins SMTP/email settings.
         */
    }

    always {

        echo 'Cleaning up temporary Docker resources...'

        sh '''
            docker image prune -f || true
        '''
    }
}
```

}
