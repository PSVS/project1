pipeline {
    agent any
    
    environment {
        WEBAPP_DIR = '/'
        DEV_BRANCH = 'psv-branch'
        GITHUB_TOKEN = credentials('github-token') // Ensure you have configured this credential in Jenkins
    }
    
    stages {
        stage('Webapp Sourcecode Checkout') {
            agent {
                node {
                    label 'functionality1'
                }
            }
            steps {
                script {
                    def res = sh(script: "sudo test -d ${WEBAPP_DIR}/panchayatseva-webapp && echo '1' || echo '0'", returnStdout: true).trim()
                    if (res == '0') {
                        sh "sudo -s cd ${WORKSPACE} && sudo mkdir ${WEBAPP_DIR} || echo webappdir exists"
                        sh "sudo -s cd ${WEBAPP_DIR} && sudo git clone -b ${DEV_BRANCH} --single-branch https://devops-sayukth:${GITHUB_TOKEN}@github.com/PanchayatSeva-SB/panchayatseva-webapp.git"
                    } else {
                        def changes_in_webapp = sh(script: "sudo -s cd ${WEBAPP_DIR}/panchayatseva-webapp && sudo git fetch && git rev-list HEAD..origin/${DEV_BRANCH} --count", returnStdout: true).trim()
                        def changes_in_devops = sh(script: "sudo -s cd ${WEBAPP_DIR}/panchayatseva-devops && sudo git fetch && git rev-list HEAD..origin/${DEV_BRANCH} --count", returnStdout: true).trim()
                        if (changes_in_webapp.toInteger() > 0 || changes_in_devops.toInteger() > 0) {
                            echo 'Changes detected. Continue with the build.'
                            sh "sudo -s cd ${WEBAPP_DIR}/panchayatseva-webapp && sudo git pull"
                        } else {
                            currentBuild.result = 'ABORTED'
                            error 'Pipeline aborted: No new changes in the repository.'
                        }
                    }
                }
            }
        }
        stage('Artifact Install') {
            agent {
                node {
                    label 'fucntionality1'
                }
            }
            steps {
                sh "sudo -s cd ${WEBAPP_DIR}/panchayatseva-webapp && /opt/apache-maven-3.9.7/bin/mvn install package -DskipTests=true -Dtestng.dtd.http=true -Dsonar.skip=true -Dsnyk.skip=true"
            }
        }
    }
}

