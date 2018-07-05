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
        stage('Clone workspace') {
            git url: 'https://github.com/rhoban/workspace.git'
        }
        stage('prepare') {
            steps {
                sh prepare.sh
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
                    junit "workspace/build_release/**/test_results/**/*.xml"
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
