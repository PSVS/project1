pipeline {
    agent any
    environment {
        GITHUB_TOKEN = credentials('github-token')
        WEBAPP_DIR = '/'
        DEV_BRANCH = 'fucntionality1' // Change to 'master' if your branch name is 'master'
    }
    stages {
        stage('Webapp Sourcecode Checkout') {
            steps {
                script {
                    def res = sh(script: "test -d ${WEBAPP_DIR}/panchayatseva-webapp && echo '1' || echo '0'", returnStdout: true).trim()
                    if (res == '0') {
                        sh "mkdir -p ${WEBAPP_DIR} || echo webappdir exists"
                        sh "cd ${WEBAPP_DIR} && git clone -b ${DEV_BRANCH} --single-branch https://devops-sayukth:$GITHUB_TOKEN@github.com/PanchayatSeva-SB/panchayatseva-webapp.git"
                    } else {
                        def changes_in_webapp = sh(script: "cd ${WEBAPP_DIR}/panchayatseva-webapp && git fetch && git rev-list HEAD..origin/${DEV_BRANCH} --count", returnStdout: true).trim()
                        def changes_in_devops = sh(script: "cd ${WEBAPP_DIR}/panchayatseva-devops && git fetch && git rev-list HEAD..origin/${DEV_BRANCH} --count", returnStdout: true).trim()
                        if (changes_in_webapp.toInteger() > 0 || changes_in_devops.toInteger() > 0) {
                            echo 'Changes detected. Continue with the build.'
                            sh "cd ${WEBAPP_DIR}/panchayatseva-webapp && git pull"
                        } else {
                            currentBuild.result = 'ABORTED'
                            error 'Pipeline aborted: No new changes in the repository.'
                        }
                    }
                }
            }
        }
        stage('Artifact Install') {
            steps {
                sh "cd ${WEBAPP_DIR}/panchayatseva-webapp && /opt/apache-maven-3.9.7/bin/mvn install package -DskipTests=true -Dtestng.dtd.http=true -Dsonar.skip=true -Dsnyk.skip=true"
            }
        }
    }
}
