pipeline {
    agent any
    stages {
        stage('Copy secrets file') {
            steps {
                script {
                    echo 'Copy secrets file ...'
                    sh 'scp -F /var/lib/jenkins/.ssh/ pi@192.168.1.91:/media/pi/Elements/homebridge-config/config.json ./config.json'
                }
            }
        }
        stage('Copy code to nfs share') {
            when {
                expression {
                    env.BRANCH_NAME == 'master'
                }
            }
            steps {
                echo 'Copy code to nfs share ...'
                sh 'scp -F /var/lib/jenkins/.ssh/  -r ${PWD}/config.json pi@192.168.1.91:/media/pi/Elements/k8sNFS/homebridge/config.json'
            }
        }
        stage('Deploy to k3s') {
            when {
                expression {
                    env.BRANCH_NAME == 'master'
                }
            }
            agent {
                docker {
                    image 'registry-192.168.1.38.nip.io/homeassistant/helm-kubectl:latest'
                    args '-v /var/lib/jenkins/config:/var/lib/jenkins/config'
                }
            }
            steps {
                    echo 'Deploying using helm...'
                    sh 'export KUBECONFIG=/var/lib/jenkins/config && helm upgrade homebridge ./homebridge/ -i -n homebridge -f ./homebridge/values.yaml --recreate-pods'
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}