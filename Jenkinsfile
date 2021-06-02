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
      parallel {
        stage ('Scanning with Check Point Shifleft') {
          steps {
            script {      
              try {
                /* Check Point shiftleft. There are some issue the scan return kernel panic */
                sh 'chmod +x shiftleft' 
                sh './shiftleft code-scan -s .'
              } catch (Exception e) {
                  echo "Security Test Failed" 
                  env.flagError = "true"  
              }
            }
          }
        }
        stage ('Scanning with SonarQube') {
          steps {
            echo "===========Performing Sonar Scan============"
            //sh "docker run -d --name sonarqube -p 9000:9000 -v /home/seong/sonarqube/data:/opt/sonarqube/data -v /home/seong/sonarqube/extensions:/opt/sonarqube/extensions sonarqube"
            sh "${tool("sonarqube")}/bin/sonar-scanner"
          }
        }
        stage ('Scanning with Fortify on Demand') {
          steps {
            echo "===========Performing Fortify On Demand Scanning============"
                  //This is Fortify on Demand, which will take around 20 mins to upload and complete scan
                  /*
                  fodStaticAssessment([
                    bsiToken: '${JUICE_SHOP_BSI_TOKEN}', 
                    entitlementPreference: 'SubscriptionFirstThenSingleScan', 
                    inProgressBuildResultType: 'FailBuild', 
                    inProgressScanActionType: 'Queue', 
                    overrideGlobalConfig: false,
                    releaseId: '151797', 
                    remediationScanPreferenceType: 'RemediationScanIfAvailable', 
                    srcLocation: "${WORKSPACE}",  
                    personalAccessToken: '',  
                    tenantId: '', 
                    username: ''
                  ]) */
          }
        }
      }
		} 
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
	  stage ("Dynamic Analysis - DAST") {
      parallel {
        stage ('OWASP ZAP') {
          steps {
            echo "Waiting app to get ready!!"
            sleep(10)
            sh "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://192.168.255.248/ -a -j || true"
		      }
        }
        stage('Arachni') {
          steps {
            echo "Waiting app to get ready!!"
            sleep(10)
            sh '''
              mkdir -p $PWD/reports $PWD/artifacts;
              docker run \
                -v $PWD/reports:/arachni/reports ahannigan/docker-arachni \
                bin/arachni http://192.168.255.248/ --checks=* --scope-directory-depth-limit=1 --scope-page-limit=1 \
                --http-user-agent="Mozilla/5.0 (X11; Linux i686; rv:6.0) Gecko/20100101 Firefox/6.0" \
                --report-save-path=reports/juice-shop.afr;
              docker run --name=arachni_report  \
                -v $PWD/reports:/arachni/reports ahannigan/docker-arachni \
                bin/arachni_reporter reports/juice-shop.afr --reporter=html:outfile=reports/juice-ship-report.html.zip;
              docker cp arachni_report:/arachni/reports/juice-shop-report.html.zip $PWD/artifacts;
              docker rm arachni_report;
            '''
            archiveArtifacts artifacts: 'artifacts/**', fingerprint: true
          }
        }
      }
  	}  
  } 
}