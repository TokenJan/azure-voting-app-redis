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
                        withKubeConfig([credentialsId: 'kubernetes',
                            serverUrl: 'https://jan-k8s-dns-2c96c686.hcp.southeastasia.azmk8s.io:443',
                            caCertificate: '-----BEGIN CERTIFICATE-----\nMIIExzCCAq+gAwIBAgIQRicfujME+979TO3sL5f0QzANBgkqhkiG9w0BAQsFADAN\nMQswCQYDVQQDEwJjYTAeFw0xOTA5MDUwMjEwMjdaFw00OTA4MjgwMjIwMjdaMA0x\nCzAJBgNVBAMTAmNhMIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEApgSk\nX8YAdAR1j9eMOyPEtbfBfAG9qJZfhX2pdCPk1+NCuOxS2KcRbbqvaqRKR/Mt+ZS2\nbInqDOFQReW+6VfrbQaBh5irdy3rKZsPICjm6i9iqECOA5+EeQKu6J1vCqzgKdWw\naOx2vY5Ez1FCMP/+IcHX+oXUKxCqBmqwS7/U1AF5Azh+vwQKiE3+eO1EahYUL1Tl\nd/yOac6pvCLC1Gh8SH3PzlWQCTo//jo8j11Bl6MvRyTSVb5zPDOViE522umuvSdh\nql9B9AUQvTp1X23PpG6D5CTfbuQ5yyCWuUhHv9/GaUgHCeG3Q9jCt3riLRjR1Bru\n5wBu772OaKjAPU1dRbndf/TtHeaBkZAZpUK/B42wo2n4HETf2SvrXK7OAmMYKqy1\npexwk4KsfO0dPaqgzAFBlkvGfuWuNLR7qZ9GTFcmAcbaf/Ro/P2dxW2T7JTQUitX\nQhI49rVfehFFRauChGbdWuClawgKngST3SeXJlrx4nArKin3UxHniqIE+JU80j3x\nne2Gc4ydSdQ0cOt4wcGJ7Sv56dg3cWCWjtAaNwyN0H41l1A3PiEMfk1rpgSTtF6w\niXz7HgqQhGIYMgwFXWsYqBlArL5KpgMB6AWe8aOi8o7HouOnPF6G7FOkDrp4jo9h\nVz2rDnGC0kM4txN4FbrlUAmwJrCVF4LTYYVIYaECAwEAAaMjMCEwDgYDVR0PAQH/\nBAQDAgKkMA8GA1UdEwEB/wQFMAMBAf8wDQYJKoZIhvcNAQELBQADggIBAJ9g/8Ut\nSr/NF47tU4u+3YtOEh4PfQyyn3QGdf147cN9T6Ff5fHvBDUN9liMdqTuggm3dd5J\ninBkzJs8NmYWOcvMRkuGHQwB3u5nAI0XXKaH+wjXbZKbEUYUzeEhWVUycC0UBWMh\nEJ7QRbMWWk5xyFEYeYbbOMV3zkG2r1zfDaBKLfk4fl0KaMB0WZ/p4CR+Ue5yxqb1\nIb2yewkgHlnCrwBR6JTUWfbRJQ95uuCiMshymyGhxkxFxRvh4lq2BCOaJVqxYjHC\nuDWP995RrzOI4x7QML4tot5fWTaeEEvfmXMPgNJRMK2YOWXITfQp0ohoJCYsuYSW\n2HFDB21iqO5lG6x4mtxSfWmSOwvypHmTww6rUYbxHbs7ZfzvdJVk+Up3wzAVOR/k\n4hdLblurKLsJ3l7ermmuY6GaQW71E0oQnKB0+Rq9vlgb1N4fOrQExZhdaGkUgI3y\nXbEOge5eFieP8iJV7zjrPx23VVh8CqkIwFtLUWOZ7HUCEzXTQu0jKL3LQm/EdpFX\nDjQmBsAIvNyUiXOyTVLIrm3qdyOx3GCDL/R+LtPNOnbRV64VHCBcRW06pJvRwqH+\nh6hUgHcb9DxOoG8UiD4lWNB3OLaPQzxScI7z3eSC1qDJbBwE2VO7PmgPtmOWlPcd\nHe1gWblGEMQsQkNgsemhepXzY4wzkdY1wbFd\n-----END CERTIFICATE-----',
                            contextName: 'jan-k8s',
                            clusterName: 'jan-k8s',
                            namespace: 'default'
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
