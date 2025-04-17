pipeline {
    agent any

    environment {
        BUILD_CONTAINER_IMAGE = 'myapp-build:latest'
        TEST_CONTAINER_IMAGE = 'myapp-test:latest'
        DEPLOY_CONTAINER_IMAGE = 'myapp-deploy:latest'
        VERSION = "1.0.${BUILD_NUMBER}"
    }

    stages {
        stage('Clear') { 
            steps {
                script{
                    sh '''
                        if [ "$(docker ps -aq)" ]; then
                          docker rm -f $(docker ps -aq)
                        fi
                        if [ "$(docker images -aq)" ]; then
                          docker rmi -f $(docker images -aq)
                        fi
                    '''
                }
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker build -t $BUILD_CONTAINER_IMAGE -f Dockerfile.build .'
            }
        }

        stage('Run Build') {
            steps {
                sh 'docker run --rm --name build-container $BUILD_CONTAINER_IMAGE | tee build.log'
            }
        }

        stage('Build Test Image') {
            steps {
                sh 'docker build -t $TEST_CONTAINER_IMAGE -f Dockerfile.test .'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'docker run --rm --name test-container $TEST_CONTAINER_IMAGE | tee test.log'
            }
        }

        stage('Save Logs as Artifact') {
            steps {
                archiveArtifacts artifacts: '*.log', fingerprint: true
            }
        }

        stage('Build Deploy Image') {
            steps {
                sh 'docker build -t $DEPLOY_CONTAINER_IMAGE -f Dockerfile.deploy .'
            }
        }

        stage('Run Deploy Container') {
            steps {
                sh 'docker run -d --name deploy-container -p 3000:3000 $DEPLOY_CONTAINER_IMAGE'
                sleep 15
            }
        }

        stage('Smoke Test') {
            steps {
                sh '''
                    echo "[TEST] Weryfikacja działania aplikacji (smoke test)..."
                    docker run --rm --network container:deploy-container appropriate/curl -s -o /dev/null -w "%{http_code}" http://localhost:3000 > status.txt
                    STATUS=$(cat status.txt)
                    if [ "$STATUS" -ne 200 ]; then
                    echo "Smoke test failed! App returned status $STATUS"
                    exit 1
                    else
                    echo "Smoke test passed! App responded with 200 OK"
                    fi
                '''
            }
        }

        stage('Debug Credentials') {
            steps {
                script {
                    def creds = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
                        com.cloudbees.plugins.credentials.common.StandardUsernameCredentials.class,
                        Jenkins.instance,
                        null,
                        null
                    )
                    echo "Dostępne poświadczenia:"
                    creds.each {
                        echo " - ID: ${it.id}, Description: ${it.description}, Username: ${it.username}"
                    }
                }
            }
        }

        stage('Publish Docker Image') {
            steps {
             withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh '''
                        echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin

                        # Tagowanie obrazu jako 'latest' i wersjonowany
                        docker tag $DEPLOY_CONTAINER_IMAGE $DOCKERHUB_USER/myapp:latest
                        docker tag $DEPLOY_CONTAINER_IMAGE $DOCKERHUB_USER/myapp:$VERSION

                        # Wysyłka na DockerHub
                        docker push $DOCKERHUB_USER/myapp:latest
                        docker push $DOCKERHUB_USER/myapp:$VERSION
                    '''
                }
            }
        }


    }

    post {
        always {
            sh 'docker rm -f deploy-container || true'
            sh 'docker logout'
        }
    }
}