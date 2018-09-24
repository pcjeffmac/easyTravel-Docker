node {
    environment {
        ET_APM_SERVER_DEFAULT = "APM"
    }
 
  	stage('cleanup') {
 		deleteDir()
 		//checkout scm
 	}
   
    stage('docker-compose') {
    	dir ('deploy-easytravel') {
    	    sh 'cd /var/lib/jenkins/jobs/easyTravelDockerPipeline/workspace/easytravel-docker'
    		step([$class: 'DockerComposeBuilder', dockerComposeFile: 'docker-compose.yml', option: [$class: 'StartAllServices'], useCustomDockerComposeFile: false])
    	}
    }
}    