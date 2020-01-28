pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "testsaccount/thoughtworks-task"
        STAGING_REPLICAS = 0
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/nodejs-app.zip'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-user') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Deploy to Staging') {
            environment {
                STAGING_REPLICAS = 1
            }
            steps {
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'nodejs-app-kube-staging.yml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('Canary Deploy') {
            environment {
                CANARY_REPLICAS = 1
            }
            steps {
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'nodejs-app-kube-canary.yml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('SmokeTest') {
            steps {
                script {
                    sleep (time: 5)
                    def response = httpRequest (
                        url: "http://$KUBE_MASTER_IP:8081/",
                        timeout: 30
                    )
                    if (response.status != 200) {
                        error("Smoke test against canary deployment failed.")
                    }
                }
            }
        }
        stage('Deploy To Production') {
            environment {
                CANARY_REPLICAS = 0
            }
            steps {
                input 'It passed the smokeTest! Do you want to proceed to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'nodejs-app-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'nodejs-app-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
    post {
        cleanup {
            kubernetesDeploy(
                kubeconfigId: 'kubeconfig',
                configs: 'nodejs-app-kube-staging.yml',
                enableConfigSubstitution: true
            )
        }
    }
}
