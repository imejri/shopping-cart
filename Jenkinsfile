pipeline {
agent { node { label 'maven' } }
environment { APP_NAMESPACE = 'security-scans' } 
stages {
        stage('Test') {
            steps {
                
                sh './mvnw clean test'
            }      
        }

        stage('Image Build') {
            steps {
               
                    sh """
                        oc start-build security-scans \
                        --follow --wait -n ${APP_NAMESPACE}
                    """ 
            }
        } 

        stage('Deploy') {
            steps {
                    sh """
                        oc patch deployment security-scans \
                        -p '{"spec": {"template": {"metadata": {"labels":
                    {"build": "build-${BUILD_NUMBER}"}}}}}' \
                        -n ${APP_NAMESPACE}
                    """
                }
        }  // stage
    } // stages
} // pipeline