pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credential')    
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '7'))
        disableConcurrentBuilds()
        timeout (time: 1, unit: 'MINUTES')
        timestamps()
    }
    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: '')
    }
    stages {
        stage('Cloning Repository') {
            steps {
                dir("${WORKSPACE}/a1innocent") {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "*/${params.BRANCH_NAME}"]],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [[$class: 'LocalBranch']],
                        submoduleCfg: [],
                        userRemoteConfigs: [[
                            url: 'git@github.com:tinno100/do-it-yourself.git',
                            credentialsId: 'github-credential'
                        ]]
                    ])
                }
            }
        }

        stage('Unit Test UI Code') {
            agent {
                docker {
                    image 'maven:3.8.5-openjdk-17'
                    args '-u root -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            environment {
                DOCKER_HOST = 'unix:///var/run/docker.sock'
            }
            steps {
                script {
                    dir('ui') {
                        sh '''
                        ls
                        mvn test
                        '''
                    }
                }
            }
        }

        stage('SonarQube analysis') {
            agent {
                docker {
                    image 'sonarsource/sonar-scanner-cli:10.0'
                }
            }
            environment {
                CI = 'true'
                scannerHome = '/opt/sonar-scanner'
            }
            steps {
                withSonarQubeEnv('Sonar') { 
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
    }
}
