pipeline {
agent {
node {
label 'slave'
    }
  }
environment {
    DOCKERHUB_CREDENTIALS = credentials('dockerhub')
    }
stages {
    stage('NOTIFICATION-EMAIL') {
        steps {
           sh 'echo pipeline started'
	   emailext body: 'this is to notify that build job has been started',
	   subject: 'jenkins-notification',
	   to: 'mteja4433@gmail.com',
	   attachLog: true
	   sh 'echo email sent'
      }
    }
    stage('CODE ANALYSIS-SONARQUBE') {
        steps {
	   sh 'echo sonar analysis started'
	   sh ' cd webapp' 
	   sh '''sudo docker run  --rm -e SONAR_HOST_URL="http://35.154.142.248:9000" -e SONAR_LOGIN="sqp_37445f8d7f977255722dd6723c02180aab1710f1" -v ".:/usr/src" sonarsource/sonar-scanner-cli:5.0 -Dsonar.projectKey=lms'''
           sh 'echo sonar analysis completed'
      }
    }
    stage('BUILD FOR ARTIFACTS') {
        steps {
           sh 'echo building LMS frontend... '
	   sh 'cd /lms/webapp'
	   sh 'npm install && npm run build'
	   sh 'echo frontend build completed'
	   sh 'cd /lms/api'
	   sh 'npm install && npm run build'
	   sh 'echo api build complete'
      }
    }
    stage('RELEASE LMS Frontend') {
        steps {
	   script {
                    echo "Releasing.."       
                    def packageJSON = readJSON file: 'webapp/package.json'
                    def packageJSONVersion = packageJSON.version
                    echo "${packageJSONVersion}"  
                    sh "zip webapp/dist-${packageJSONVersion}.zip -r webapp/dist"
                    sh "curl -v -u admin:amkamk3@ --upload-file webapp/dist-${packageJSONVersion}.zip http://65.1.132.50:8081/repository/lmsfe/"  
		    sh 'echo lmsfe pushed to nexus repo'   
            }
      }
    }
    stage('RELEASE LMS-Backend') {
        steps {
	   script {
                    echo "Releasing.."       
                    def packageJSON = readJSON file: 'webapp/package.json'
                    def packageJSONVersion = packageJSON.version
                    echo "${packageJSONVersion}"  
                    sh "zip webapp/dist-${packageJSONVersion}.zip -r webapp/dist"
                    sh "curl -v -u admin:amkamk3@ --upload-file webapp/dist-${packageJSONVersion}.zip http://65.1.132.50:8081/repository/lmsfe/"  
		    sh 'echo lmsbe pushed to nexus repo'   
            }
      }
    }

    stage('Docker Build') {
        steps {
	   sh 'cd /lms/api'
	   sh 'docker build -t Chocolate123/api .'
	   sh 'cd /lms/webapp'
	   sh 'docker build -t chocolate123/webapp .'
	   sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
	   sh 'docker push chocolate123/api'
	   sh 'docker push chocolate123/webapp'
           sh 'echo docker images build and pushed to docker hub'
      }
    }
    stage('APPROVAL-SLACK/EMAIL') {
        steps {
           sh 'echo requsting approval to deployment'
	               stage ('Approval') {
                steps {
                    emailext     subject: """Deployment Approval for $deployBranch branch and $deployService service""",
                               	        body: '''<a href="${JENKINS_URL}/job/${JOB_NAME}/${BUILD_NUMBER}/input">click to approve</a>''',
                                to: 'muralialakuntla3@gmail.com',
                                mimeType: 'text/html',
                                attachLog: true
                    script {
                        def Delay = input id: 'Deploy',
                            message: sh(script:'''echo "You are DEPLOYING -->$deployBranch<-- IN PRODUCTION"''', returnStdout: true).trim(),
                            submitter: 'uaserID1, userID2',
                            parameters: [
                                    [$class: 'ChoiceParameterDefinition',
                                     choices: ['no','yes'].join('\n'),
                                     name: 'input',
                                     description: 'Please Select "YES" to Build or "NO" to Abort']
                            ]
                            echo "The answer is: ${Delay}"
                            if( "${Delay}" == "yes"){
                            sh'''echo "Deploying in prod"'''
                            } else {
                            sh """
                            echo "exiting not production ready branch"
                            exit 1
                            """
                            }
                    }
                }
            }

	   
      }
    }
stage('approval-slack') {
    steps {
        script {
            timeout(time: 5, unit: 'MINUTES') {
                    slackSend(
                    channel: 'team-updates',
                    message: "Approval needed for deployment: ${env.JOB_NAME} (${env.JENKINS_URL}/job/${env.JOB_NAME}/${env.BUILD_NUMBER}/console)",
                    teamDomain: 'konalms',
                    tokenCredentialId: 'slack'
                )
                  input message: 'Approve to Deploy?', ok: 'Yes'
            }
        }
    }
}

    stage('DEPLOYMENT-K8S') {
        steps {
           sh 'echo application deployment completed'
      }
    }
  }
}
