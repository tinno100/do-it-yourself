pipeline {
    agent {
        label 'master'
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '7'))
        // skipDefaultCheckout(true)
        disableConcurrentBuilds()
        timeout (time: 1, unit: 'MINUTES')
        timestamps()
    }
    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: '')
        string(name: 'DB_IMAGE_VERSION', defaultValue: 'develop', description: '')
        string(name: 'REDIS_IMAGE_VERSION', defaultValue: 'develop', description: '')
        string(name: 'UI_IMAGE_VERSION', defaultValue: 'develop', description: '')
        string(name: 'WEATHER_IMAGE_VERSION', defaultValue: 'develop', description: '')
        string(name: 'AUTH_IMAGE_VERSION', defaultValue: 'develop', description: '')
    }
    stages {
        stage ('Checkout') {
            steps {
                dir("${WORKSPACE}/code") {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "*/${params.BRANCH_NAME}"]],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [[$class: 'LocalBranch']],
                        submoduleCfg: [],
                        userRemoteConfigs: [[
                        url: 'https://github.com/devopstia/a1.git',
                        credentialsId: 'github-auth'
                        ]]
                    ])
                }
            }
        }
        stage('Open the compose file') {
            steps {
                script {
                    dir("${WORKSPACE}/code/jenkins/docker-compose") {
                        sh """
                            cat docker-compose.yaml
                        """
                    }
                }
            }
        }
        // stage('Set variables') {
        //     steps {
        //         script {
        //             dir("${WORKSPACE}/code/jenkins/docker-compose") {
        //                 settingUpVariable()
        //             }
        //         }
        //     }
        // }
        stage('Set variables') {
            steps {
                script {
                     dir("${WORKSPACE}/code/jenkins/docker-compose") {
                        withCredentials([
                        usernamePassword(credentialsId: 'weather-app-redis-cred', 
                        usernameVariable: 'REDIS_USERNAME', 
                        passwordVariable: 'REDIS_PASSWORD'),

                        string(credentialsId: 'weather-app-api-key', 
                        variable: 'API_TOKEN'),

                        string(credentialsId: 'weather-app-mysql-root-password', 
                        variable: 'MYSQL_ROOT_PASSWORD'),

                        string(credentialsId: 'weather-app-mysql-password', 
                        variable: 'MYSQL_PASSWORD')]) {
                            settingUpVariable()
                        }
                     }
                }
            }
        }
        stage('Pull Images') {
            steps {
                script {
                    dir("${WORKSPACE}/code/jenkins/docker-compose") {
                        pullImages()
                    }
                }
            }
        }
    }
}



def settingUpVariable() {
    sh """
    sed -i "s|DB_IMAGE_VERSION|${params.DB_IMAGE_VERSION}|g" docker-compose.yaml
    sed -i "s|REDIS_IMAGE_VERSION|${params.REDIS_IMAGE_VERSION}|g" docker-compose.yaml
    sed -i "s|UI_IMAGE_VERSION|${params.UI_IMAGE_VERSION}|g" docker-compose.yaml
    sed -i "s|WEATHER_IMAGE_VERSION|${params.WEATHER_IMAGE_VERSION}|g" docker-compose.yaml
    sed -i "s|AUTH_IMAGE_VERSION|${params.AUTH_IMAGE_VERSION}|g" docker-compose.yaml
    
    sed -i "s|WEATHER_APP_REDIS_PASSWORD_USERNAME|${REDIS_USERNAME}|g" docker-compose.yaml
    sed -i "s|WEATHER_APP_REDIS_PASSWORD|${REDIS_PASSWORD}|g" docker-compose.yaml
    sed -i "s|WEATHER_API-TOKEN|${API_TOKEN}|g" docker-compose.yaml
    sed -i "s|WEATHER_APP_MYSQL_ROOT_PASSWORD|${MYSQL_ROOT_PASSWORD}|g" docker-compose.yaml
    sed -i "s|WEATHER_APP_MYSQL_PASSWORD|${MYSQL_PASSWORD}|g" docker-compose.yaml
    cat docker-compose.yaml
    """
}

def pullImages() {
    sh """
    sudo docker pull devopseasylearning/sixfure-db:${params.DB_IMAGE_VERSION}
    sudo docker pull devopseasylearning/sixfure-redis:${params.REDIS_IMAGE_VERSION}
    sudo docker pull devopseasylearning/sixfure-ui:${params.UI_IMAGE_VERSION}
    sudo docker pull devopseasylearning/sixfure-weather:${params.WEATHER_IMAGE_VERSION}
    sudo docker pull devopseasylearning/sixfure-auth:${params.AUTH_IMAGE_VERSION}
    sudo docker images
    sudo docker-compose down
    sudo docker-compose up -d
    sleep 15
    sudo docker-compose ps
    """
}

// sed -i "s|DB_IMAGE_VERSION|develop|g" docker-compose.yaml
// sed -i "s|REDIS_IMAGE_VERSION|develop|g" docker-compose.yaml
// sed -i "s|UI_IMAGE_VERSION|develop|g" docker-compose.yaml
// sed -i "s|WEATHER_IMAGE_VERSION|develop|g" docker-compose.yaml
// sed -i "s|AUTH_IMAGE_VERSION|develop|g" docker-compose.yaml

// cat docker-compose.yaml




