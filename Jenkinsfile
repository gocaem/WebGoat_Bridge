pipeline {
    agent any
    tools {
        maven 'maven-3.9'
        jdk 'openjdk-17'
    }
    environment {
        REPO_NAME = "${env.GIT_URL.tokenize('/.')[-2]}"
        //BRIDGECLI_LINUX64 = 'https://sig-repo.synopsys.com/artifactory/bds-integrations-release/com/synopsys/integration/synopsys-bridge/latest/synopsys-bridge-linux64.zip'
        //BRIDGE_POLARIS_SERVERURL = 'https://poc.polaris.synopsys.com'
        BRIDGE_POLARIS_APPLICATION_NAME = "gonz_test-${env.REPO_NAME}"
        BRIDGE_POLARIS_PROJECT_NAME = "gonz_test-${env.REPO_NAME}"
        BRIDGE_POLARIS_ASSESSMENT_TYPES = 'SAST,SCA'
    }
   stages {
   //     stage('Build') {
   //         steps {
   //             sh 'mvn -B package'
   //         }
   //     }
        stage('polaris') {
            steps {
                withCredentials([string(credentialsId: 'polaris_token', variable: 'BRIDGE_POLARIS_ACCESSTOKEN')]) {
                    script {
                        status = sh returnStatus: true, script: '''
                            curl -fLsS -o bridge.zip $BRIDGECLI_LINUX64 && unzip -qo -d $WORKSPACE_TMP bridge.zip && rm -f bridge.zip
                            $WORKSPACE_TMP/synopsys-bridge --verbose --stage polaris \
                                polaris.project.name=$BRIDGE_POLARIS_PROJECT_NAME \
                                polaris.branch.name=$BRANCH_NAME \
                                polaris.application.name=$BRIDGE_POLARIS_APPLICATION_NAME \
                                polaris.assessment.types=$BRIDGE_POLARIS_ASSESSMENT_TYPES \
                                polaris.serverurl=$BRIDGE_POLARIS_SERVERURL
                        '''
                        if (status == 8) { unstable 'policy violation' }
                        else if (status == 2) { unstable 'Synopsys Bridge received a non-0 exit code from an internal adapter. Review the log file for details.' }
                        else if (status != 0) { error 'scan failure' }
                    }
                }
            }
        }
    }
  //  post {
  //      always {
            //zip archive: true, dir: '.bridge', zipFile: 'bridge-logs.zip'
            //cleanWs()
      //  }
  //  }
}
