@Library('jenkins-shared-library@main') _
pipeline {

  agent any
  
  parameters {
	choice(name: 'action', choices: 'create\nrollback', description: 'Create/rollback of the deployment')
    string(name: 'ImageName', description: "Name of the docker build", defaultValue: "docker")
	string(name: 'ImageTag', description: "Name of the docker build",defaultValue: "v1")
	string(name: 'AppName', description: "Name of the Application",defaultValue: " ")
    string(name: 'docker_repo', description: "Name of docker repository",defaultValue: "bharanitallaplly")
  }
      
  tools{ 
        maven 'maven3'
    }
    stages {
        stage('Git-Checkout') {
            when {
				expression { params.action == 'create' }
			}
            steps {
                gitCheckout(
                    branch: "main",
                    url: "https://github.com/bharanitallapally/spring-cloud-kubernetes.git"
                )
            }
        }
        stage('Build-Maven'){
            when {
				expression { params.action == 'create' }
			}
    		steps {
        		dir("${params.AppName}") {
        			sh 'mvn clean package'
        		}
    		}
	    }
	    stage("DockerBuild and Push") {
	        when {
				expression { params.action == 'create' }
			}
	        steps {
	            dir("${params.AppName}") {
	                dockerBuild ( "${params.ImageName}", "${params.docker_repo}" )
	            }
	        }
	    }
	    stage("Docker-CleanUP") {
	        when {
				expression { params.action == 'create' }
			}
	        steps {
	            dockerCleanup ( "${params.ImageName}", "${params.docker_repo}" )
			}
		}
			    stage("Ansible Setup") {
	        when {
				expression { params.action == 'create' }
			}
	        steps {
	            sh 'ansible-playbook ${WORKSPACE}/kubernetes-configmap-reload/server_setup.yml'
			}
		}
	    stage("Create deployment") {
			when {
				expression { params.action == 'create' }
			}
	        steps {
	            sh 'echo ${WORKSPACE}'
	            sh 'kubectl create -f ${WORKSPACE}/kubernetes-configmap-reload/kubernetes-configmap.yml'
	        }
	    }
	    stage ("wait_for_pods"){
	    steps{
              
                sh 'sleep 300'
             
	    }
	    }
		stage("rollback deployment") {
	        steps {	            	         	           
	              sh """
	                   kubectl delete deploy ${params.AppName}
					    kubectl delete svc ${params.AppName}
				  """	       
	        }
	    }
    }
}
