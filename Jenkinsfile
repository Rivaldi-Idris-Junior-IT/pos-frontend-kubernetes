def builderDocker
def CommitHash

pipeline {
    agent any

    parameters {
        booleanParam(name: 'RUNTEST', defaultValue: true, description: 'Toggle this value for testing')
        choice(name: 'Deploy', choices: ['production', 'deployement'], description: 'Deploy Other Server')
        choice(name: 'CICD', choices: ['CI', 'CICD'], description: 'Pick something')
        choice(name: 'Mode', choices: ['master', 'production'], description: 'Pili mode push')
    }

    stages {
        stage('Build Project') {
            steps {
                nodejs("node12") {
                    sh 'npm install'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    if (params.Mode == GIT_BRANCH) {
                        script {
                            CommitHash = sh(script: "git log -n 1 --pretty=format:'%H'", returnStdout: true)
                            builderDocker = docker.build("aldifarzum/dockerpos-frontend:master")
                        }
                        sh 'echo Validasi branch berhasil'
                    } else if (params.Mode != GIT_BRANCH) {
                        currentBuild.result = 'ABORTED'
                        error('Validasi branch gagal …')
                    }
                }
            }
        }

        stage('Run Testing') {
            when {
                expression {
                    params.RUNTEST
                }
            }
            steps {
                script {
                    builderDocker.inside {
                        sh 'echo passed'
                    }
                }
            }
        }

        stage('Push Image') {
            when {
                expression {
                    params.RUNTEST
                }
            }
            steps {
                script {
                    builderDocker.push("${env.GIT_BRANCH}")
                }
            }
        }



        stage('Deploy-deployement') {
            when {
                expression {
                    params.CICD == 'CICD'
                }
            }
            steps {
                script {
                    if (params.Deploy == 'deployement') {
                        sshPublisher(
                            publishers: [
                                sshPublisherDesc(
                                    configName: 'Development',
                                    verbose: false,
                                    transfers: [
                                        sshTransfer(
                                            execCommand: 'docker pull aldifarzum/dockerpos-frontend:master; docker run -d --rm --name frontend -p 8080:80 aldifarzum/dockerpos-frontend:master',
                                            execTimeout: 120000,
                                        )
                                    ]
                                )
                            ]
                        )
                    } else if (params.Deploy == 'production') {
                        sshPublisher(
                            publishers: [
                                sshPublisherDesc(
                                    configName: 'Production',
                                    verbose: false,
                                    transfers: [
                                        sshTransfer(
                                            execCommand: 'docker pull aldifarzum/dockerpos-frontend:master; docker kill master; docker run -d --rm --name frontend -p 8080:80 aldifarzum/dockerpos-frontend:master',
                                            execTimeout: 120000,
                                        )
                                    ]
                                )
                            ]
                        )
                    }
                }
            }


        }
    }
}