node {
    environment {
        ET_APM_SERVER_DEFAULT = "APM"
    }
 
  	stage('cleanup') {
 		deleteDir()
 		//checkout scm
 	}
 
    stage('Checkout') {
        dir ('easytravel-docker') {
            git url: 'https://github.com/pcjeffmac/easyTravel-Docker.git', credentialsId: '0ab85f6f2492796b11f0fdb1cded9efe37e0a68e', branch: 'master'
        }
    }
    
    stage('docker-compose') {
    	dir ('deploy-easytravel') {
    	    sh 'cd /var/lib/jenkins/jobs/easyTravelDockerPipeline/workspace/easytravel-docker'
    		step([$class: 'DockerComposeBuilder', dockerComposeFile: 'docker-compose.yml', option: [$class: 'StartAllServices'], useCustomDockerComposeFile: true])
    	}
    }
}    