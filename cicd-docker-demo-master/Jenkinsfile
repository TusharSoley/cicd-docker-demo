pipeline {
    agent any
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
                    app = docker.build("rijuvijayan/train-schedule")
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToStaging') {
            when {
                branch 'master'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "ssh -i /var/lib/jenkins/.ssh/id_rsa $USERNAME@$staging_ip \"docker pull rijuvijayan/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "ssh -i /var/lib/jenkins/.ssh/id_rsa $USERNAME@$staging_ip \"docker stop train-schedule\""
                            sh "ssh -i /var/lib/jenkins/.ssh/id_rsa $USERNAME@$staging_ip \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "ssh -i /var/lib/jenkins/.ssh/id_rsa $USERNAME@$staging_ip \"docker run --restart always --name train-schedule -p 8080:8080 -d rijuvijayan/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
        stage('DeployToProd') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "ssh -i /var/lib/jenkins/.ssh/id_rsa $USERNAME@$prod_ip \"docker pull rijuvijayan/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "ssh -i /var/lib/jenkins/.ssh/id_rsa $USERNAME@$prod_ip \"docker stop train-schedule\""
                            sh "ssh -i /var/lib/jenkins/.ssh/id_rsa $USERNAME@$prod_ip \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "ssh -i /var/lib/jenkins/.ssh/id_rsa $USERNAME@$prod_ip \"docker run --restart always --name train-schedule -p 9001:8080 -d rijuvijayan/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
