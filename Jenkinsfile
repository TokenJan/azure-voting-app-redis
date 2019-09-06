pipeline {
    agent any
    environment {
        GITVERSION   = 'gittools/gitversion:5.0.0-linux-centos7-netcoreapp2.1'
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
                sh 'docker build -t ${ACR_LOGINSERVER}/azure-vote-front:latest azure-vote/'
                sh 'docker tag ${ACR_LOGINSERVER}/azure-vote-front:latest ${ACR_LOGINSERVER}/azure-vote-front:$(docker run --rm --volume "$(pwd):/repo" $GITVERSION /repo -output json -showvariable FullSemVer)'
                sh 'docker login -u $ACR_USER -p $ACR_PASSWORD $ACR_LOGINSERVER'
                sh 'docker push ${ACR_LOGINSERVER}/azure-vote-front:$(docker run --rm --volume "$(pwd):/repo" $GITVERSION /repo -output json -showvariable FullSemVer)'
                sh 'docker push ${ACR_LOGINSERVER}/azure-vote-front:latest'
                sh 'docker rmi ${ACR_LOGINSERVER}/azure-vote-front:latest ${ACR_LOGINSERVER}/azure-vote-front:$(docker run --rm --volume "$(pwd):/repo" $GITVERSION /repo -output json -showvariable FullSemVer)'
            }
        }

        stage('deploy:dev') {
            parallel {
                stage('deploy') {
                    steps {
                        withKubeConfig([
                            credentialsId: 'kubernetes',
                            serverUrl: 'https://jan-k8s-dns-2c96c686.hcp.southeastasia.azmk8s.io:443'
                        ]) {
                            sh 'kubectl set image deployment azure-vote-front azure-vote-front=tokenjanacr.azurecr.io/azure-vote-front:$(docker run --rm --volume "$(pwd):/repo" $GITVERSION /repo -output json -showvariable FullSemVer)'
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