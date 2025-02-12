pipeline {
    agent any
    environment {
        VERSION   = '4.0'
    }
    stages {
        stage('pre-build') {
            parallel {
                stage('style check') {
                    steps {
                        echo 'style check'
                    }
                }
                stage('TSLint') {
                    steps {
                        echo 'TSLint'
                    }
                }
                stage('find bugs') {
                    steps {
                        echo 'find bugs'
                    }
                }
                stage('unit test') {
                    steps {
                        echo 'unit test'
                    }
                }
            }
        }

        stage('build and publish') {
            steps {
                echo 'build'
                sh 'docker login -u $ACR_USER -p $ACR_PASSWORD $ACR_LOGINSERVER'
                sh 'docker build -t ${ACR_LOGINSERVER}/azure-vote-front:${VERSION} azure-vote/'
                sh 'docker push ${ACR_LOGINSERVER}/azure-vote-front:${VERSION}'
            }
        }
        
        stage('deploy:dev') {
            parallel {
                stage('deploy') {
                    steps {
                        withKubeConfig([
                            credentialsId: 'ec-aks-prod',
                            serverUrl: 'https://ec-prod-k8s-dns-3482a302.hcp.japanwest.azmk8s.io'
                        ]) {
                            sh 'kubectl set image deployment azure-vote-front azure-vote-front=${ACR_LOGINSERVER}/azure-vote-front:${VERSION} -n default'
                        }
                    }
                }
                stage('contract test') {
                    steps {
                        echo 'contract test'
                    }
                }
                stage('sonarqube scanning') {
                    steps {
                        echo 'sonarqube scanning'
                    }
                }
            }
        }
    }
}
