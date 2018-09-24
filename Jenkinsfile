node {
    environment {
        ET_APM_SERVER_DEFAULT = "APM"
    }
        
    stage('docker-down') {
    	sh 'cd /var/lib/jenkins/jobs/easyTravelDockerPipeline/workspace/deploy-easytravel'
    	step([$class: 'DockerComposeBuilder', dockerComposeFile: '/var/lib/jenkins/jobs/easyTravelDockerPipeline/workspace/deploy-easytravel/docker-compose.yml', option: [$class: 'StopAllServices'], useCustomDockerComposeFile: true])
    } 
 
  	stage('cleanup') {
 		//deleteDir()
 		//checkout scm
 	}
   
    stage('docker-compose') {
    	dir ('deploy-easytravel') {
    		sh '''cd /var/lib/jenkins/jobs/easyTravelDockerPipeline/workspace@script
				cp -R * ../workspace/deploy-easytravel/. 
				cp .env ../workspace/deploy-easytravel/.'''
    		step([$class: 'DockerComposeBuilder', dockerComposeFile: '/var/lib/jenkins/jobs/easyTravelDockerPipeline/workspace/deploy-easytravel/docker-compose.yml', option: [$class: 'StartAllServices'], useCustomDockerComposeFile: true])
    	}
    }
    
    stage('networking') {
    	sh 'sudo su -'
    	sh 'iptables -t nat -A POSTROUTING --source 172.17.0.6 --destination 172.17.0.6 -p tcp --dport 80 -j MASQUERADE'
		sh 'iptables -t nat -A DOCKER ! -i docker0 --source 0.0.0.0/0 --destination 0.0.0.0/0 -p tcp --dport 80  -j DNAT --to 172.17.0.6:80'
		sh 'iptables -A DOCKER ! -i docker0 -o docker0 --source 0.0.0.0/0 --destination 172.17.0.6 -p tcp --dport 80 -j ACCEPT'
    }
    
}    