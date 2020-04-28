def TEST_START = 'UNKNOWN'
def TEST_END = 'UNKNOWN'

pipeline {
agent any
    environment {
        ET_APM_SERVER_DEFAULT = "APM"
    }

stages {
 	stage('cleanup') {
 	    steps {
 		deleteDir()
		}
 	}

	stage('Flush iptables') {
	    steps {
		sh '/home/dynatrace/resetiptables.sh'
		}
	}

    stage('Checkout-cli') {
        steps {
        // Checkout our application source code
            git url: 'https://github.com/pcjeffmac/easyTravel-Docker.git', credentialsId: "${GIT_TOKEN}", branch: 'master'
        // into a dynatrace-cli subdirectory we checkout the CLI
        dir ('dynatrace-cli') {
            git url: 'https://github.com/Dynatrace/dynatrace-cli.git', credentialsId: "${GIT_CLI_TOKEN}", branch: 'master'
        }
        }
    }
          
   stage('docker-down') {
        steps {
    	sh 'cd /var/lib/jenkins/jobs/easyTravelDockerPipeline/workspace/'
    	step([$class: 'DockerComposeBuilder', dockerComposeFile: '/var/lib/jenkins/jobs/easyTravelDockerPipeline/workspace/docker-compose.yml', option: [$class: 'StopAllServices'], useCustomDockerComposeFile: true])
    	}
    } 
   
   stage('docker-compose-up') {
        steps {
    	dir ('deploy-easytravel') {
    		sh 'cd /var/lib/jenkins/jobs/easyTravelDockerPipeline/workspace/'
    		step([$class: 'DockerComposeBuilder', dockerComposeFile: '/var/lib/jenkins/jobs/easyTravelDockerPipeline/workspace/docker-compose.yml', option: [$class: 'StartAllServices'], useCustomDockerComposeFile: true])
    	}
    	}
   }
    
   stage('Event-Post-Host') {
   		steps {
        	//Dynatrace POST action for deployment Event  
        	script {     	
        	jsonPayload = """{"eventType": "CUSTOM_DEPLOYMENT",
  					"attachRules": {
    				"tagRule" : {
        			"meTypes" : "HOST",
        				"tags" : "easyTravelDocker"
    					}
  					},
  					"deploymentName":"${JOB_NAME} - ${BUILD_NUMBER} Staging (http)",
  					"deploymentVersion":"1.1",
  					"deploymentProject":"easyTravelDocker",
  					"remediationAction":"https://ansible.pcjeffint.com/#/templates/job_template/7",
  					"ciBackLink":"${BUILD_URL}",
  					"source":"Jenkins",
  					"customProperties":{
    					"Jenkins Build Number": "${BUILD_ID}",
    					"Environment": "Production",
    					"Job URL": "${JOB_URL}",
    					"Build URL": "${BUILD_URL}"
  						}
					}"""
				}	
		    //echo "${jsonPayload}"			
        	//send json payload	
			httpRequest (acceptType: 'APPLICATION_JSON', 
			authentication: 'a47386bc-8488-41c0-a806-07b1123560e3', 
			contentType: 'APPLICATION_JSON', 
			customHeaders: [[maskValue: true, name: 'Authorization', 
			value: "Api-Token ${DT_API_TOKEN}"]], 
			httpMode: 'POST', 
			ignoreSslErrors: true, 
			requestBody: jsonPayload, 
			responseHandle: 'NONE', 
			url: "${DT_TENANT_URL}/api/v1/events/",
			validResponseCodes: '200')        			
		}
    }    

    stage('Event-Post-Service-JourneyService') {
        steps {
            script {
        	//Dynatrace POST action for deployment Event      	
        	jsonPayload = """{"eventType": "CUSTOM_DEPLOYMENT",
  					"attachRules": {
    				"tagRule" : {
        			"meTypes" : "SERVICE",
        				"tags" : "JourneyService"
    					}
  					},
  					"deploymentName":"${JOB_NAME} - ${BUILD_NUMBER} Staging (http)",
  					"deploymentVersion":"1.1",
  					"deploymentProject":"easyTravelDocker",
  					"remediationAction":"https://ansible.pcjeffint.com/#/templates/job_template/7",
  					"ciBackLink":"${BUILD_URL}",
  					"source":"Jenkins",
  					"customProperties":{
    					"Jenkins Build Number": "${BUILD_ID}",
    					"Environment": "Production",
    					"Job URL": "${JOB_URL}",
    					"Build URL": "${BUILD_URL}"
  						}
					}"""
				}		
        	//send json payload	
			httpRequest (acceptType: 'APPLICATION_JSON', 
			authentication: 'a47386bc-8488-41c0-a806-07b1123560e3', 
			contentType: 'APPLICATION_JSON', 
			customHeaders: [[maskValue: true, name: 'Authorization', 
			value: "Api-Token ${DT_API_TOKEN}"]], 
			httpMode: 'POST', 
			ignoreSslErrors: true, 
			requestBody: jsonPayload, 
			responseHandle: 'NONE', 
			url: "${DT_TENANT_URL}/api/v1/events/",
			validResponseCodes: '200')		
		}        		
    } 
    
    stage('Event-Post-Service-Nginx') {
        steps {
            script {
        	//Dynatrace POST action for deployment Event      	
        	jsonPayload = """{"eventType": "CUSTOM_DEPLOYMENT",
  					"attachRules": {
    				"tagRule" : {
        			"meTypes" : "SERVICE",
        				"tags" : "etNginx"
    					}
  					},
  					"deploymentName":"${JOB_NAME} - ${BUILD_NUMBER} Staging (http)",
  					"deploymentVersion":"1.1",
  					"deploymentProject":"easyTravelDocker",
  					"remediationAction":"https://ansible.pcjeffint.com/#/templates/job_template/7",
  					"ciBackLink":"${BUILD_URL}",
  					"source":"Jenkins",
  					"customProperties":{
    					"Jenkins Build Number": "${BUILD_ID}",
    					"Environment": "Production",
    					"Job URL": "${JOB_URL}",
    					"Build URL": "${BUILD_URL}"
  						}
					}"""
			}
        	//send json payload	
			httpRequest (acceptType: 'APPLICATION_JSON', 
			authentication: 'a47386bc-8488-41c0-a806-07b1123560e3', 
			contentType: 'APPLICATION_JSON', 
			customHeaders: [[maskValue: true, name: 'Authorization', 
			value: "Api-Token ${DT_API_TOKEN}"]], 
			httpMode: 'POST', 
			ignoreSslErrors: true, 
			requestBody: jsonPayload, 
			responseHandle: 'NONE', 
			url: "${DT_TENANT_URL}/api/v1/events/",
			validResponseCodes: '200')
		}      		
    }    

    stage('networking-rules') {
        steps {
    	sh 'docker inspect --format \'{{ .NetworkSettings.IPAddress }}\' www'
    	sh 'export DWWW=`docker inspect --format \'{{ .NetworkSettings.IPAddress }}\' www`'	
		script {
			DWWW = sh (script: 'docker inspect --format \'{{ .NetworkSettings.IPAddress }}\' www', returnStdout: true).trim()
		}	
		echo "This is the ip set: ${DWWW}"		
    		sh "sudo iptables -t nat -A POSTROUTING --source ${DWWW} --destination ${DWWW} -p tcp --dport 80 -j MASQUERADE"
			sh "sudo iptables -t nat -A DOCKER ! -i docker0 --source 0.0.0.0/0 --destination 0.0.0.0/0 -p tcp --dport 80  -j DNAT --to ${DWWW}"
			sh "sudo iptables -A DOCKER ! -i docker0 -o docker0 --source 0.0.0.0/0 --destination ${DWWW} -p tcp --dport 80 -j ACCEPT"
		}
    }  
  
   stage('Run NeoLoad - scenario1') {
       steps {

        script {
        TEST_START = sh(script: 'echo "$(date -u +%s)000"', returnStdout: true).trim()
        }

         //step(
          // dir ('NeoLoad') {
        
           //PerfSig record test
    		//recordDynatraceSession(entityIds: [[$class: 'Service', entityId: 'SERVICE-2A07FD2D00BA8372']], envId: 'DTSaaS', testCase: 'loadtest')
    		//{
    	      // Test scenario
    	      //NeoLoad Test 
    		  //neoloadRun executable: "${NL_CMD_PATH}", 
    		  //project: "${NL_PROJECT}", 
    		  //testName: 'scenerio1' + '$Date{hh:mm - dd MMM yyyy}' + "(build ${BUILD_NUMBER})", 
    		  //testDescription: 'From Jenkins', 
    		  //commandLineOption: "-nlweb -nlwebAPIURL ${NL_WEB_URL} -nlwebToken ${NL_WEB_TOKEN} -noGUI", 
    		  //scenario: 'scenario1', trendGraphs: ['AvgResponseTime', 'ErrorRate']     
			//}      
           //}
         //)

        script {
        TEST_END = sh(script: 'echo "$(date -u +%s)000"', returnStdout: true).trim()
        } 
       
        }
    }
    
   stage('Annotation-Post') {
       steps {
        echo "${TEST_START}"	
        echo "${TEST_END}"	       
       		script {
        	//Dynatrace POST action for deployment Event      	
        	jsonPayload = """{"eventType": "CUSTOM_ANNOTATION",
  					"attachRules": {
    				"tagRule" : {
        			"meTypes" : "SERVICE",
        				"tags" : "etNginx"
    					}
  					},
  					"source":"Jenkins",
  					"annotationType": "LoadTest",
  		            "annotationDescription": "NeoLoad Run",
  					"customProperties":{
    					"Jenkins Build Number": "${BUILD_ID}",
    					"Environment": "Production"
  						},
  						"start": ${TEST_START},
  						"end": ${TEST_END} 
					}"""
        	}
        	//send json payload	
			httpRequest (acceptType: 'APPLICATION_JSON', 
			authentication: 'a47386bc-8488-41c0-a806-07b1123560e3', 
			contentType: 'APPLICATION_JSON', 
			customHeaders: [[maskValue: true, name: 'Authorization', 
			value: "Api-Token ${DT_API_TOKEN}"]], 
			httpMode: 'POST', 
			ignoreSslErrors: true, 
			requestBody: jsonPayload, 
			responseHandle: 'NONE', 
			url: "${DT_TENANT_URL}/api/v1/events/",
			validResponseCodes: '200')
		}      		
    }      
    
    stage('ValidateProduction') {
       steps {
        script {
        dir ('dynatrace-scripts') {
            DYNATRACE_PROBLEM_COUNT = sh (script: './checkforproblems.sh', returnStatus : true)
            echo "Dynatrace Problems Found: ${DYNATRACE_PROBLEM_COUNT}"
        }
        }

		//Produce PerSig reports
        perfSigDynatraceReports (envId: 'DTSaaS', 
        nonFunctionalFailure: 2, 
        specFile: '/var/lib/jenkins/jobs/easyTravelDockerPipeline/workspace/monspec/monspec.json')
        

        // now lets generate a report using our CLI and lets generate some direct links back to dynatrace
        dir ('dynatrace-cli') {
            sh 'python3 dtcli.py dqlr srv tags/CONTEXTLESS:easyTravelDocker=www '+
               'service.responsetime[avg%hour],service.responsetime[p90%hour] ${DT_URL} ${DT_TOKEN}'
            sh 'mv dqlreport.html dqlproductionreport.html'
            archiveArtifacts artifacts: 'dqlproductionreport.html', fingerprint: true

            sh 'python3 dtcli.py link srv tags/CONTEXTLESS:easyTravelDocker=www ' +
            	' overview 60:0 ${DT_URL} ${DT_TOKEN} > dtprodlinks.txt'
            archiveArtifacts artifacts: 'dtprodlinks.txt', fingerprint: true
        }
        
        }
    }     
}

post {
    	success {
      		slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    	}
    
    	unstable {
      		slackSend (color: '#FFFF00', message: "UNSTABLE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    	}

    	failure {
      		slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    	}
    }

}    
