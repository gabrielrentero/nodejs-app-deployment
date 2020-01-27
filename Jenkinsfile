pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "testsaccount/thoughtworks-task"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/thoughtworks.zip'
            }
        }
        stage('Build Docker Image') {
            // when {
            //     branch 'master'
            // }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            // when {
            //     branch 'master'
            // }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-user') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            // when {
            //     branch 'master'
            // }
            steps {
                input 'Do you want to deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'thoughtworks-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
