pipeline {
    agent any
    environment {
        DOCKER_REGISTRY = 'https://registry.hub.docker.com'
        DOCKER_HUB_CREDENTIALS = 'docker_hub_login'
    }
    
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
                    app = docker.build("rociomm123/train-schedule")
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
                    // Get the list of existing tags from the Docker registry
                    // Pull the Docker image to get the latest tag

                    print ("${env.BUILD_NUMBER}")
                    // def versionNumbers = tagsOutput.tokenize().findAll { it =~ /^\d+\.\d+\d+$/ }.collect { it.tokenize('/')[1].tokenize(':')[1] as Float }

                    // // Find the highest version number
                    // def currentVersion = versionNumbers.max()
                    
                    // // If there are no existing tags, start from version 0.001
                    // if (currentVersion == null) {
                    //     currentVersion = 0.001
                    // }

                    // Increment version number by 0.001 and format to three decimal places
                    // def incrementedVersion = String.format("%.3f", currentVersion + 0.001)

                    // Use the incremented version number for building and pushing the Docker image
                    docker.withRegistry("${DOCKER_REGISTRY}", "${DOCKER_HUB_CREDENTIALS}") {
                    //     // Push Docker image with the incremented version number as tag
                    //     app.push("${incrementedVersion}")
                    // docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        // app.push("latest")
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
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull rociomm123/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop train-schedule\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name train-schedule -p 8080:8080 -d rociomm123/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
