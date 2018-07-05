pipeline {
    agent {
        dockerfile {
            additionalBuildArgs  '--build-arg SSH_KEY="`cat ~/.ssh/id_rsa`" --build-arg USER_ID=`id -u` --build-arg USER_GID=`id -g`'
        }
    }
    options { disableConcurrentBuilds() }

    environment {
        QN_SLACK_MSG = sh(returnStdout: true, script: '''
            echo "Build #$BUILD_NUMBER (<$JOB_DISPLAY_URL|Voir>)"
            echo "Branch: $BRANCH_NAME"
            echo "Auteur: `git show -s --pretty=%an`"
            echo "Message: `git show -s --pretty=%s`"
            ''')
        QN_COLOR_FAIL = '#7b0d1e'
        QN_COLOR_SUCCESS = '#44cf6c'
        QN_COLOR_NORMAL= '#439FE0'
    }

    stages {
        stage('debug') {
            steps {
               sh '''
                   echo "fake message"
                   '''
            }
            post {
                failure {
                   echo "failure message"
                   '''
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
                success {
                    slackSend color: "$QN_COLOR_SUCCESS", message: "Tests validés\n\n$QN_SLACK_MSG"
                }
                failure {
                    slackSend color: "$QN_COLOR_FAIL", message: "Echec `tests`: Oh la la :grin:\n\n$QN_SLACK_MSG"
                }
            }
        }

        stage('generate docs') {
            steps {
                sh '''
                    cd workspace
                    doxygen
                '''
            }

            post {
                success {
                    slackSend color: "$QN_COLOR_SUCCESS", message: "Docs générées\n\n$QN_SLACK_MSG"
                }
                failure {
                    slackSend color: "$QN_COLOR_FAIL", message: "Echec `generate docs`: Oh la la :grin:\n\n$QN_SLACK_MSG"
                }
            }
        }

        stage('collect docs') {
            when { branch 'master' }
            steps {
                sh '''
                    cd workspace
                    tar czvf doc.tar.gz doc/html
                '''
            }

            post {
                success {
                    archive 'workspace/doc.tar.gz'
                    slackSend color: "$QN_COLOR_SUCCESS", message: "Docs collectées\n\n$QN_SLACK_MSG"
                }
                failure {
                    slackSend color: "$QN_COLOR_FAIL", message: "Echec `collect docs`: Oh la la :grin:\n\n$QN_SLACK_MSG"
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
