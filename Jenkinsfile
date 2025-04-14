pipeline {
    agent any
    stages {
        stage('Clear') { 
            steps {
                sh 'docker rm -f $(docker ps -aq)'
	            sh 'docker rmi -f $(docker images -aq)' 
            }
        }
    }
}