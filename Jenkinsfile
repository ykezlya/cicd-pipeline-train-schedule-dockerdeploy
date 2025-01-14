pipeline {
    agent { label 'docker_builder' }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("ykezlya/train-schedule")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'hub_docker') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "ssh $USERNAME@$prod_ip \"docker pull ykezlya/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "ssh $USERNAME@$prod_ip \"docker stop train-schedule\""
                            sh "ssh $USERNAME@$prod_ip \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "ssh $USERNAME@$prod_ip \"docker run --restart always --name train-schedule -p 8080:8080 -d ykezlya/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
