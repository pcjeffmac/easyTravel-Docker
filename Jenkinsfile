node {
    environment {
        ET_APM_SERVER_DEFAULT = "APM"
    }

 	stage('cleanup') {
 		deleteDir()
 	}

	stage('Flush iptables') {
		sh '/home/dynatrace/resetiptables.sh'
	}

    stage('Checkout-cli') {
        // Checkout our application source code
            git url: 'https://github.com/pcjeffmac/easyTravel-Docker.git', credentialsId: '0ab85f6f2492796b11f0fdb1cded9efe37e0a68e', branch: 'master'
        // into a dynatrace-cli subdirectory we checkout the CLI
        dir ('dynatrace-cli') {
            git url: 'https://github.com/Dynatrace/dynatrace-cli.git', credentialsId: 'cd41a86f-ea57-4477-9b10-7f9277e650e1', branch: 'master'
        }
    }
          
    stage('docker-down') {
    	sh 'cd /var/lib/jenkins/jobs/easyTravelDockerPipeline/workspace/'
    	step([$class: 'DockerComposeBuilder', dockerComposeFile: '/var/lib/jenkins/jobs/easyTravelDockerPipeline/workspace/docker-compose.yml', option: [$class: 'StopAllServices'], useCustomDockerComposeFile: true])
    } 
 
  	stage('cleanup') {
 		//deleteDir()
 		//checkout scm
 	}
   
    stage('docker-compose-up') {
    	dir ('deploy-easytravel') {
    		sh 'cd /var/lib/jenkins/jobs/easyTravelDockerPipeline/workspace/'
    		step([$class: 'DockerComposeBuilder', dockerComposeFile: '/var/lib/jenkins/jobs/easyTravelDockerPipeline/workspace/docker-compose.yml', option: [$class: 'StartAllServices'], useCustomDockerComposeFile: true])
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
        	
			httpRequest acceptType: 'APPLICATION_JSON', authentication: 'a47386bc-8488-41c0-a806-07b1123560e3', contentType: 'APPLICATION_JSON', customHeaders: [[maskValue: true, name: 'Authorization', value: 'Api-Token CGVha39QTheyn1UFufsvC']], httpMode: 'POST', ignoreSslErrors: true, requestBody: body, responseHandle: 'NONE', url: 'https://ibg73613.live.dynatrace.com/api/v1/events/'        		
    }
    
    stage('networking-rules') {
    	sh 'docker inspect --format \'{{ .NetworkSettings.IPAddress }}\' www'
    	sh 'export DWWW=`docker inspect --format \'{{ .NetworkSettings.IPAddress }}\' www`'
		
		DWWW = sh (
			script: 'docker inspect --format \'{{ .NetworkSettings.IPAddress }}\' www',
			returnStdout: true
		).trim()	
		echo "This is the ip set: ${DWWW}"
		
    		sh "sudo iptables -t nat -A POSTROUTING --source ${DWWW} --destination ${DWWW} -p tcp --dport 80 -j MASQUERADE"
			sh "sudo iptables -t nat -A DOCKER ! -i docker0 --source 0.0.0.0/0 --destination 0.0.0.0/0 -p tcp --dport 80  -j DNAT --to ${DWWW}"
			sh "sudo iptables -A DOCKER ! -i docker0 -o docker0 --source 0.0.0.0/0 --destination ${DWWW} -p tcp --dport 80 -j ACCEPT"


    }  
    
   stage('Run NeoLoad - scenario1') {
        dir ('NeoLoad') {
        //NeoLoad Test
        neoloadRun executable: '/opt/Neoload6.6/bin/NeoLoadCmd', project: '/home/dynatrace/NeoLoadProjects/easytravelDocker/easytravelDocker.nlp', testName: 'scenerio1 $Date{hh:mm - dd MMM yyyy} (build ${BUILD_NUMBER})', testDescription: 'From Jenkins', commandLineOption: '-nlweb -nlwebAPIURL http://neoload.pcjeffint.com:8080/ -nlwebToken qBmjKe13OshxpIm4npUQLNOE', scenario: 'scenario1', trendGraphs: ['AvgResponseTime', 'ErrorRate']     
        }
    }
    
    stage('ValidateProduction') {
        dir ('dynatrace-scripts') {
            DYNATRACE_PROBLEM_COUNT = sh (script: './checkforproblems.sh', returnStatus : true)
            echo "Dynatrace Problems Found: ${DYNATRACE_PROBLEM_COUNT}"
        }
   
        // now lets generate a report using our CLI and lets generate some direct links back to dynatrace
        dir ('dynatrace-cli') {
            sh 'python3 dtcli.py dqlr srv tags/CONTEXTLESS:DockerService=easyTravelDocker '+
               'service.responsetime[avg%hour],service.responsetime[p90%hour] ${DT_URL} ${DT_TOKEN}'
            sh 'mv dqlreport.html dqlproductionreport.html'
            archiveArtifacts artifacts: 'dqlproductionreport.html', fingerprint: true

            sh 'python3 dtcli.py link srv tags/CONTEXTLESS:DockerService=easyTravelDocker ' +
            	' overview 60:0 ${DT_URL} ${DT_TOKEN} > dtprodlinks.txt'
            archiveArtifacts artifacts: 'dtprodlinks.txt', fingerprint: true
        }
    }           
    
}    