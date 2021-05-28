pipeline {
  environment {
      registryCredential = "DOCKER_HUB_TOKEN"
      NGINX_REPO_CERT = credentials("NGINX_REPO_EVAL_CERT")
      NGINX_REPO_KEY = credentials("NGINX_REPO_EVAL_KEY")
      CHKP_CLOUDGUARD_ID = credentials("CHKP_CLOUDGUARD_ID")
      CHKP_CLOUDGUARD_SECRET = credentials("CHKP_CLOUDGUARD_SECRET")
      JUICE_SHOP_BSI_TOKEN = credentials("JUICE_SHOP_BSI_TOKEN")
      //FORTIFY_ON_DEMAND_TOKEN = credentials("FORTIFY_ON_DEMAND_TOKEN")
   }
  agent any
  stages {
    stage('Clone Github repository') {
        steps {
            checkout scm
            //checkout scm: ([
            //        $class: 'GitSCM',
            //        userRemoteConfigs: [[credentialsId: '',url: 'https://github.com/bkimminich/juice-shop.git']],
            //        branches: [[name: '*/master']]
            //])
        }
    }
    stage('Scan source code before containerizing App') {    
      steps {
        script {      
          try {
            //sh 'chmod +x shiftleft' 
            //sh './shiftleft code-scan -s .'
            fodStaticAssessment([
              bsiToken: '${JUICE_SHOP_BSI_TOKEN}', 
              entitlementPreference: 'SubscriptionFirstThenSingleScan', 
              inProgressBuildResultType: 'FailBuild', 
              inProgressScanActionType: 'Queue', 
              overrideGlobalConfig: false,
              releaseId: '1', 
              remediationScanPreferenceType: 'RemediationScanIfAvailable', 
              srcLocation: "${WORKSPACE}",  
              personalAccessToken: '',  
              tenantId: '', 
              username: ''
            ]) 
          } catch (Exception e) {
              echo "Security Test Failed" 
              env.flagError = "true"  
            }
          }
      }
    } /*
    stage('Code approval request') {
      when {
        expression { env.flagError == "true" }
      }
      steps {
        script {
            def userInput = input(id: 'confirm', message: 'This code contains vulnerabilities. Would you still like to continue?', parameters: [ [$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Approve Code to Proceed', name: 'approve'] ])
            env.flagError = "false"  
        }
      }
    }
    stage('Building image') {
        steps{
            sh 'cat ${NGINX_REPO_CERT} > "nginx/nginx-repo.crt"'
            sh 'cat ${NGINX_REPO_KEY} > "nginx/nginx-repo.key"'
            sh 'docker-compose build'
        }
    } 
    stage('Scan container before pushing to Dockerhub') {    
      steps {
        script {      
          try {
            sh 'docker save akhng999/juice-shop -o vwa.tar' 
            sh './shiftleft image-scan -i ./vwa.tar -t 1800'
          } catch (Exception e) {
            echo "Security Test Failed" 
            env.flagError = "true"  
          }
        }
      }
    }
    stage('Dockerhub Approval Request') {
        when {
            expression { env.flagError == "true" }
        }
        steps {
            script {
                env.PUSH_TO_DOCKER_HUB = input message: 'This containers contains vulnerabilities. Push to Dockerhub?', 
                parameters: [choice(name: 'Push to Docker Hub', choices: 'no\nyes', description: 'Choose "yes" if you want to push this build')]
            }
        }
    }
    stage('Pushing Image') {
        when {
            environment name: 'PUSH_TO_DOCKER_HUB', value: 'yes'
        }
        steps {
            echo "pushing to docker hub registry"
            script {
                docker.withRegistry('', registryCredential) {
                    sh 'docker-compose push'
                }
            }
        }
    }
    stage('Deploy the Applications') {
        steps {
            sh 'docker-compose up -d'
        }
    }
	  stage ("Dynamic Analysis - DAST with OWASP ZAP") {
        steps {
            echo "Waiting app to get ready!!"
            sleep(10)
            sh "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://192.168.255.212/ -a -j || true"
		    }
  	}  */
  } 
}