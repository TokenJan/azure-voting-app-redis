node {
    stage('style check') {
        echo 'style check'
    }
    stage('TSLint') {
        echo 'TSLint'
    }
    stage('find bugs') {
        echo 'find bugs'
    }
    stage('unit test') {
        echo 'unit test'
    }

    stage('build and publish') {
        echo 'build'
        SEMVER = sh(script: 'docker run --rm --volume "$(pwd):/repo" gittools/gitversion:5.0.0-linux-centos7-netcoreapp2.1 /repo -output json -showvariable FullSemVer', returnStdout: true)

        dir("azure-vote/") {
            sh 'docker build -t ${ACR_LOGINSERVER}/azure-vote-front:latest .'
            sh 'docker tag ${ACR_LOGINSERVER}/azure-vote-front:latest ${ACR_LOGINSERVER}/azure-vote-front:${SEMVER}'
            sh 'docker login -u $ACR_USER -p $ACR_PASSWORD $ACR_LOGINSERVER'
            sh 'docker push ${ACR_LOGINSERVER}/azure-vote-front:${SEMVER}'
            sh 'docker rmi ${ACR_LOGINSERVER}/azure-vote-front:latest ${ACR_LOGINSERVER}/azure-vote-front:${SEMVER}'
        }
    }

    stage('deploy') {
        withKubeConfig([
            credentialsId: 'kubernetes',
            serverUrl: 'https://jan-k8s-dns-2c96c686.hcp.southeastasia.azmk8s.io:443'
        ]) {
            sh 'kubectl set image deployment azure-vote-front azure-vote-front=tokenjanacr.azurecr.io/azure-vote-front:latest'
        }
    }
    stage('contract test') {
        echo 'contract test'
    }
    stage('sonarqube scanning') {
        echo 'sonarqube scanning'
    } 
}