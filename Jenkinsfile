pipeline {
    agent { label 'galileo' }
    tools {
        maven 'maven-3.9'
        jdk 'openjdk-17'
    }
    environment {
        // extract PROJECT_NAME from GIT_URL
        //PROJECT_NAME = "${env.GIT_URL.tokenize('/.')[-2]}"
        PROJECT_NAME = 'webgoat'
    }
    stages {
        stage('build') {
            steps {
                sh "mvn -B -DskipTests package"
            }
        }
        stage('blackduck') {
            steps {
                withCredentials([string(credentialsId: 'poc329.blackduck.synopsys.com', variable: 'BLACKDUCK_API_TOKEN')]) {
                    sh '''
                        docker build -f Dockerfile.detect -t detect .
                        docker run --rm -u $(id -u):$(id -g) -v $WORKSPACE:/source -v $WORKSPACE:/output detect \
                            --blackduck.url=$BLACKDUCK_URL --blackduck.api.token=$BLACKDUCK_API_TOKEN \
                            --detect.project.name=$PROJECT_NAME --detect.project.version.name=$BRANCH_NAME --detect.code.location.name=$PROJECT_NAME-$BRANCH_NAME \
                            --detect.policy.check.fail.on.severities=BLOCKER --detect.risk.report.pdf=true
                    '''
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts allowEmptyArchive: true, artifacts: '*_BlackDuck_RiskReport.pdf'
            cleanWs()
        }
    }
}
