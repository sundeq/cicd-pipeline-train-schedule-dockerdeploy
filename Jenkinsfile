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
                    app = docker.build("micsnbricks/train-schedule-cool-app")
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
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
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
              input 'Deploy to production'
              milestone(1)
              withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                  script {
                      sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker pull micsnbricks:/train-schedule-cool-app:${env.BUILD_NUMBER}\"" 
                      try {
                          sh "shpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@{env.prod_ip} \"docker stop train-schedule-cool-app\""
                          sh "shpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@{env.prod_ip} \"docker rm train-schedule-cool-app\""
                      } catch (err) {
                          echo: "caught error: $err"
                      }
                      sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker run --restart always --name train-schedule-cool-app -p 8080:8080 -d micsnbricks/train-schedule-cool-app:${env.BUILD_NUMBER}\"" 
                  }
              }
            }
        }
    }
}
