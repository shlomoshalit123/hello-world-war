pipeline {
    environment {
        nexus = "13.50.194.17:5000"
        sonarqube = "13.53.38.182:9000"
        docker_image_name = "hello-world-war"
        BUILD_STATUS='FAILED'
        RECIPIENTS='shalitshlomo@gmail.com'
    }

    agent {
        label 'ubuntu'
    }

    triggers {
        pollSCM('* * * * *')
    }

    stages {
        stage('Checkout Code') {
            steps {
                cleanWs()
                checkout scmGit(branches: [[name: '*/dev']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/shlomoshalit123/hello-world-war.git']])
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv(credentialsId: 'sonarqube-aws', installationName: 'sonarqube-aws') { // You can override the credential to be used
                     sh '''mvn clean verify sonar:sonar \
                      -Dsonar.projectKey=final-project \
                      -Dsonar.projectName='final-project' \
                      -Dsonar.host.url=http://${sonarqube} \
                      -Dsonar.token=sqp_22dbe3fe8e074d9054238746dadce8c3b8c18f15'''
                }
            }
        }
        stage('Build') {
            steps {
                sh 'cat Dockerfile'
                sh 'docker build -f Dockerfile -t "${nexus}/${docker_image_name}:${BUILD_NUMBER}" .'
            }
        }
        stage('Publish') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus', passwordVariable: 'nexus_password', usernameVariable: 'nexus_user')]) {
                sh 'docker login -u ${nexus_user} -p ${nexus_password} http://${nexus}'
                sh 'docker push ${nexus}/${docker_image_name}:${BUILD_NUMBER}'
                }
                script {
                    BUILD_STATUS = 'COMPLETED'
                }
            }
        }
    }
    post {
       always {
            sh 'echo ***** CLEANUP *****;'
            sh 'echo ***** Pipeline will delete the following images created during this run *****;'
            sh 'docker images --filter=reference="${nexus}/${docker_image_name}:${BUILD_NUMBER}" 2>/dev/null'
            sh 'docker rmi -f $(docker images --filter=reference="${nexus}/${docker_image_name}:${BUILD_NUMBER}" -q) 2>/dev/null'
            script {
                EMAIL_BODY = "Build Name: ${JOB_BASE_NAME}\nBuild ID:${BUILD_ID}\nStatus: ${BUILD_STATUS}\nLink: ${RUN_DISPLAY_URL}"
                EMAIL_SUBJECT = "BUILD ${JOB_BASE_NAME} ${BUILD_STATUS}"
            }
            mail bcc: '', body: "${EMAIL_BODY}", cc: '', from: '', replyTo: '', subject: "${EMAIL_SUBJECT}", to: "${RECIPIENTS}"
        }
    }
}
