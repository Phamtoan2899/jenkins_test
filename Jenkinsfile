// pipeline {
//     agent any
//     stages {
//         stage('Demo') {
//             steps {
//                 sh 'echo "Jenkins test"'
//             }
//         }
//     }
// }


pipeline {
    agent any
    environment {
//         PRIVATE_KEY            = credentials('privatekey')
        DOCKER_IMAGE = "phamtoan/nginx-${GIT_BRANCH.tokenize('/').pop()}"
        DOCKERHUB_CREDENTIALS = 'phamtoan'
    }
    stages {
        stage("Build") {
            options {
                timeout(time: 10, unit: 'MINUTES')
            }
            environment {
                DOCKERHUB_CREDENTIALS = credentials('phamtoan')
                DOCKER_TAG = "${GIT_BRANCH.tokenize('/').pop()}-${GIT_COMMIT.substring(0, 7)}"
            }
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                    docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                    docker image ls | grep ${DOCKER_IMAGE}'''
                withCredentials([usernamePassword(credentialsId: 'phamtoan', usernameVariable: 'DOCKER_HUB_USERNAME', passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
                    sh 'echo $DOCKER_HUB_PASSWORD | docker login --username $DOCKER_HUB_USERNAME --password-stdin'
                }

                sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                sh "docker push ${DOCKER_IMAGE}:latest"
                //clean to save disk
                sh "docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG}"
                sh "docker image rm ${DOCKER_IMAGE}:latest"
                sh "echo ${GIT_BRANCH}"
            }
        }

        stage("Deploy main Branch Tasks") {
            options {
                timeout(time: 10, unit: 'MINUTES')
            }
            when { expression {env.GIT_BRANCH == 'origin/main'}}
            steps {
                withCredentials([usernamePassword(credentialsId: 'phamtoan', usernameVariable: 'DOCKER_HUB_USERNAME', passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
                    ansiblePlaybook(
                            credentialsId: '18.162.152.221',
                            playbook: 'playbook.yml',
                            inventory: 'hosts',
                            become: 'yes',
                            extraVars: [
                                    DOCKER_HUB_USERNAME: "${DOCKER_HUB_USERNAME}",
                                    DOCKER_HUB_PASSWORD: "${DOCKER_HUB_PASSWORD}"
                            ]
                    )
                }
            }
        }

        stage("Deploy Develop Branch Tasks") {
            options {
                timeout(time: 10, unit: 'MINUTES')
            }
            when { expression {env.GIT_BRANCH == 'origin/dev'}}
            steps {
                withCredentials([usernamePassword(credentialsId: 'phamtoan', usernameVariable: 'DOCKER_HUB_USERNAME', passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
                    ansiblePlaybook(
                            credentialsId: '18.162.152.221',
                            playbook: 'develop/playbook.yml',
                            inventory: 'hosts',
                            become: 'yes',
                            extraVars: [
                                    DOCKER_HUB_USERNAME: "${DOCKER_HUB_USERNAME}",
                                    DOCKER_HUB_PASSWORD: "${DOCKER_HUB_PASSWORD}"
                            ]
                    )
                }
            }
        }
    }
}