node {
    environment {
        ET_APM_SERVER_DEFAULT = "APM"
    }

    stage('Checkout-cli') {
        // into a dynatrace-cli subdirectory we checkout the CLI
        dir ('dynatrace-cli') {
            git url: 'https://github.com/Dynatrace/dynatrace-cli.git', credentialsId: 'cd41a86f-ea57-4477-9b10-7f9277e650e1', branch: 'master'
        }
    }
          
    stage('docker-down') {
    	sh 'cd /var/lib/jenkins/jobs/easyTravelDockerPipeline/workspace/'
    	step([$class: 'DockerComposeBuilder', dockerComposeFile: '/var/lib/jenkins/jobs/easyTravelDockerPipeline/workspace/deploy-easytravel/docker-compose.yml', option: [$class: 'StopAllServices'], useCustomDockerComposeFile: true])
    } 
 
  	stage('cleanup') {
 		//deleteDir()
 		//checkout scm
 	}
   
    stage('docker-compose-up') {
    	dir ('deploy-easytravel') {
    		sh 'cd /var/lib/jenkins/jobs/easyTravelDockerPipeline/workspace/'
    		step([$class: 'DockerComposeBuilder', dockerComposeFile: '/var/lib/jenkins/jobs/easyTravelDockerPipeline/workspace/deploy-easytravel/docker-compose.yml', option: [$class: 'StartAllServices'], useCustomDockerComposeFile: true])
    	}
        	//Dynatrace POST action for deployment Event      	
        	def body = """{"eventType": "CUSTOM_DEPLOYMENT",
  					"attachRules": {
    				"tagRule" : {
        			"meTypes" : "HOST",
        				"tags" : "easyTravelDocker"
    					}
  					},
  					"deploymentName":"${JOB_NAME} - ${BUILD_NUMBER} Staging (http)",
  					"deploymentVersion":"1.1",
  					"deploymentProject":"easyTravelDocker",
  					"remediationAction":"https://192.168.2.85/#/templates/job_template/7",
  					"ciBackLink":"${BUILD_URL}",
  					"source":"Jenkins",
  					"customProperties":{
    					"Jenkins Build Number": "${BUILD_ID}",
    					"Environment": "Production",
    					"Job URL": "${JOB_URL}",
    					"Build URL": "${BUILD_URL}"
  						}
					}"""
        	
			httpRequest acceptType: 'APPLICATION_JSON', authentication: 'a47386bc-8488-41c0-a806-07b1123560e3', contentType: 'APPLICATION_JSON', customHeaders: [[maskValue: true, name: 'Authorization', value: 'Api-Token 7tEzakG8S2-02dv5w8SU2']], httpMode: 'POST', ignoreSslErrors: true, requestBody: body, responseHandle: 'NONE', url: 'https://buh931.dynatrace-managed.com/e/89c9109a-79f9-43c7-8f78-37372eca07e1/api/v1/events/'        		
    }
    
    stage('networking-rules') {
    	sh 'docker inspect --format \'{{ .NetworkSettings.IPAddress }}\' www'
    	sh 'export DWWW=`docker inspect --format \'{{ .NetworkSettings.IPAddress }}\' www`'
		withEnv(['DWWW=`docker inspect --format \\\'{{ .NetworkSettings.IPAddress }}\\\' www`\'']) {
    		sh 'sudo iptables -t nat -A POSTROUTING --source ${DWWW} --destination ${DWWW} -p tcp --dport 80 -j MASQUERADE'
			sh 'sudo iptables -t nat -A DOCKER ! -i docker0 --source 0.0.0.0/0 --destination 0.0.0.0/0 -p tcp --dport 80  -j DNAT --to ${DWWW}'
			sh 'sudo iptables -A DOCKER ! -i docker0 -o docker0 --source 0.0.0.0/0 --destination ${DWWW} -p tcp --dport 80 -j ACCEPT'
		}
    }
    
   stage('Run NeoLoad - scenario1') {
        dir ('NeoLoad') {
        //NeoLoad Test
        neoloadRun executable: '/opt/Neoload6.6/bin/NeoLoadCmd', project: '/home/dynatrace/NeoLoadProjects/easytravelDocker/easytravelDocker.nlp', testName: 'scenerio1 $Date{hh:mm - dd MMM yyyy} (build ${BUILD_NUMBER})', testDescription: 'From Jenkins', commandLineOption: '-nlweb -nlwebAPIURL http://192.168.3.3:8080/ -nlwebToken UYWWcrEsfg5o37ASFBdeXh9Y', scenario: 'scenario1', trendGraphs: ['AvgResponseTime', 'ErrorRate']     
        }
    }     
    
}    