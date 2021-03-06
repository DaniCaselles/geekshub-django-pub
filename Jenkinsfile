pipeline {
    /*
    *  agent {
    *    node {
    *      label 'DEBIAN8'
    *    }
    *  }
    */
    agent any
    triggers {
        pollSCM('* * * * */1')
    }
    options {
        disableConcurrentBuilds()
    }
    environment {
        registry = "danicli/geekshub-django"
        registryCredential = 'Docker'
        apiServer = "https://192.168.64.3:8443"
        devNamespace = "default"
        minikubeCredential = 'minikube-auth-token'
        imageTag = "${env.GIT_BRANCH + '_' + env.BUILD_NUMBER}"
    }
    stages {
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Build image') {
            steps {
                script {
                    dockerImage = docker.build(registry + ":$imageTag","--network host .")
                }
            }
        }
        stage('Upload to registry') {
            steps{
                script {
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Deploy to K8s') {
            steps{
                withKubeConfig([credentialsId: minikubeCredential,
                                serverUrl: apiServer,
                                namespace: devNamespace
                               ]) {
                    sh 'kubectl set image deployment/django django="$registry:$imageTag" --record'
                }
            }
        }
    }
    /* post {
    *    success {
    *        script {
    *            showtime.completeDeployment(this)
    *        }
    *    }
    *    failure {
    *        script {
    *            if(env.GIT_BRANCH == productionBranch) {
    *                def String slackChannel = "#hf-platform-alerts"
    *
    *                slackSend baseUrl: slackUrl, 
    *                channel: slackChannel, 
    *                color: '#951D13', 
    *                message: "$env.GIT_URL: Error deploying branch $env.GIT_BRANCH", 
    *                teamDomain: 'nw-hf', 
    *                tokenCredentialId: slackTokenId
    *
    *                showtime.abortDeployment(this)
    *            }
    *        }
    *    }
    *}
    */
}