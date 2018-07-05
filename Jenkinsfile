pipeline {
    agent {
        dockerfile {
            additionalBuildArgs  '--build-arg SSH_KEY="`cat ~/.ssh/id_rsa`" --build-arg USER_ID=`id -u` --build-arg USER_GID=`id -g`'
        }
    }
    options { disableConcurrentBuilds() }

    environment {
        QN_COLOR_FAIL = '#7b0d1e'
        QN_COLOR_SUCCESS = '#44cf6c'
        QN_COLOR_NORMAL= '#439FE0'
    }

    stages {
        stage('debug') {
            steps {
               echo "fake message"
            }
            post {
                failure {
                   echo "failure message"
                }
            }
        }
        stage('prepare') {
            steps {
                sh '''
                    git clone https://github.com/rhoban/workspace.git
                    cd workspace    
                    ./workspace setup
                    cd src
                    mkdir rhobandeps
                    git clone  https://github.com/rhobandeps/jsoncpp.git rhobandeps/jsoncpp
                    git clone  https://github.com/rhobandeps/eigen.git rhobandeps/eigen
                    mkdir rhoban
                    git clone  https://github.com/rhoban/utils.git rhoban/utils
                '''
            }
            post {
                failure {
                echo "Failed to prepare"
                }
            }
        }
        stage('build') {
            steps {
                sh '''
                    cd workspace
                    ./workspace build
                    ./workspace build_tests
                '''
           }
        }
        stage('tests') {
            steps {
                sh '''
                    cd workspace
                    ./workspace run_tests
                    ./workspace result_tests
                '''
            }

            post {
                always {
                    junit 'workspace/build_release/**/test_results/**/*.xml'
                }
            }
        }
    }
    post {
        always {
            deleteDir()
        }
    }
}
