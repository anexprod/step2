pipeline {
    agent { label 'jenkins_worker' }

    stages {
        stage('Pull Code') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: 'master']], userRemoteConfigs: [[url: 'git@github.com:anexprod/step2.git', credentialsId: 'GIT_SSH_KEY']]])
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
                
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build('anexprod21/step2')
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    def app = docker.image('anexprod21/step2')
                    app.inside('--entrypoint="" -v /home/vagrant/opt/jenkins/workspace/step2-test-pipeline:/app -w /app') {
                        sh 'npm test'
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            when {
                expression {
                    currentBuild.result == null || currentBuild.result == 'SUCCESS'
                }
            }
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker_log_pass') {
                        def customImage = docker.image('anexprod21/step2')
                        customImage.push('latest')
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Build and tests succeeded'
        }
        failure {
            echo 'Tests failed'
        }
    }
}