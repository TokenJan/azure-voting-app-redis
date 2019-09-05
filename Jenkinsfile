properties([
           parameters([
                   string(name: 'URL', defaultValue: 'https://github.com/TokenJan/azure-voting-app-redis.git'),
                   string(name: 'BRANCH', defaultValue: 'master'),
           ])
])

pipeline {
    agent any
    environment {
        GITVERSION   = 'gittools/gitversion:5.0.0-linux-debian-9-netcoreapp2.2'
    }
    stages {
        stage('checkout') {
            steps {
                checkout ( [$class: 'GitSCM',
                branches: [[name: '${BRANCH}' ]],
                userRemoteConfigs: [[
                credentialsId: '5c73d461-8fee-4434-9f63-26e45e845e69', 
                url: '${URL}']]])
            }
        }
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

        stage('build') {
            steps {
                echo 'build'
                sh 'docker-compose up --build -d'
                sh 'docker tag azure-vote-front ${ACR_LOGINSERVER}/azure-vote-front:$(docker run --rm --volume "$(pwd):/repo" $GITVERSION /repo -output json -showvariable FullSemVer)'
            }
        }
        stage('publish') {
            steps {
                sh 'docker login -u $ACR_USER -p $ACR_PASSWORD $ACR_LOGINSERVER'
                sh 'docker push ${ACR_LOGINSERVER}/azure-vote-front:$(docker run --rm --volume "$(pwd):/repo" $GITVERSION /repo -output json -showvariable FullSemVer)'
            }
        }
        stage('deploy:dev') {
            parallel {
                stage('deploy') {
                    steps {
                        sh 'kubectl set image deployment azure-vote-front azure-vote-front=tokenjanacr.azurecr.io/azure-vote-front:$(docker run --rm --volume "$(pwd):/repo" $GITVERSION /repo -output json -showvariable FullSemVer)'
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
