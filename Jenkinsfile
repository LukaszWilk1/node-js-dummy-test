pipeline {
    agent any
    stages {
        stage('Clear') { 
            steps {
                script{
                    sh '''
                        rm -rf /path/to/folder/* /path/to/folder/.[!.]*

                        if [ "$(docker ps -aq)" ]; then
                          docker rm -f $(docker ps -aq)
                        fi
                        if [ "$(docker images -aq)" ]; then
                        fi
                    '''
                }
            }
        }
    }
}