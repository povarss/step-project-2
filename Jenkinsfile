pipeline {
    agent { label 'jenkins-worker' }

    environment {
        GIT_URL = "https://github.com/povarss/step-project-2.git"
        GIT_BRANCH = "main"

        DOCKER_IMAGE = 'node-app'
        DOCKER_HUB_REPO = 'povarss/my-node-app'
        DOCKER_CREDENTIALS_ID = 'docker-hub-token'

        TEST_SUCCESS = "false"
    }

    stages {
        stage('Pull Code') {
            steps {
                script {
                    git url: "${GIT_URL}",
                        branch: "${GIT_BRANCH}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $DOCKER_IMAGE .'
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    try {
                        sh "docker run --rm ${DOCKER_IMAGE} test"
                        currentBuild.description = "Tests Passed"
                        TEST_SUCCESS = "true"
                        echo 'Tests passed successfully'
                    } catch (Exception e) {
                        currentBuild.description = "Tests Failed"
                        TEST_SUCCESS = "false"
                        echo 'Tests failed'
                        error("Test failed")
                    }

                    echo "TEST_SUCCESS value is: ${TEST_SUCCESS}"
                }
            }
        }

        stage('Push to Docker Hub') {
            when {
                expression {
                    echo "TEST_SUCCESS value is: ${TEST_SUCCESS}"
                    return TEST_SUCCESS == "true"
                }
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                            docker tag $DOCKER_IMAGE $DOCKER_HUB_REPO:latest
                            docker push $DOCKER_HUB_REPO:latest
                        """
                        echo 'Image pushed to Docker Hub successfully'
                    }
                }
            }
        }
    }
}
