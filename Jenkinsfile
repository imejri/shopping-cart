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

        stage('Security Scan') {
            steps {
                sh """
                oc process -f kubefiles/security-scan-template.yml \
                -n ${APP_NAMESPACE} \
                -p PROJECT_NAME=${APP_NAMESPACE} \
                -p CVE_CODE=CVE-2020-14583 \
                | oc replace --force -n ${APP_NAMESPACE} -f -
                """
                sh "! ../tools/check-job-state.sh security-scans-trivy ${APP_NAMESPACE}"
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