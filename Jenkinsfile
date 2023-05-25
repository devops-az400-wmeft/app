pipeline {
    agent any

    tools {
    	nodejs 'nodeJSv18'
	jdk 'jdkv17'
	maven 'maven'
	ansible 'ansible'
    }
    
    environment {
    	CRYPTOGRAPHY_ALLOW_OPENSSL_102=1
    }

    stages {
        stage ('Clone proxyCommerce ') {
            steps{
                script {
                    git(
                            url: 'https://github.com/eoxis/proxyCommerce.git', 
                            credentialsId: '66b94ca1-e0ef-4c7e-bf9d-560703e97812', 
                            branch: "main"
                        )
                
                }
            }
        }

        stage('Configure access to database'){
            steps{
	    	script{
			sh 'export CRYPTOGRAPHY_ALLOW_OPENSSL_102=1'
			sh 'ls docker/ansible'
			
		}
                dir('docker')
                {
                    ansiblePlaybook(
                        playbook: 'ansible/config_replace.yaml',
			installation: 'ansible',
                        colorized: true
                    )
                }
            }
        }

        stage ('Install Dependencies for proxyCommerce Front') {
            agent{
                docker {
                    image 'node:17-alpine'
                    args '-p 3000:3000 -w "/home" -v "proxy-front:/home"'
                }
            }
            steps {
               // dir("home"){
                    sh 'npm install'
              //  }
            }
        }

        stage('Build proxyCommerce front') {
            steps {
                dir("proxy-front") {
                    sh 'ng build '

                }
            }
        }

        stage ('build proxyCommerce front docker image') {
            steps {
                dir('Docker')
                {
                    sh 'docker build -f proxy-front/Dockerfile -t proxy-front'
                    sh 'docker image save -o proxy-front.tar proxy-front'
                    ansiblePlaybook(
                        vaultCredentialsId: 'ansible-vault',
                        inventory: 'ansible/inv',
                        playbook: 'ansible/copy_proxy_front_docker_image.yaml',
			installation: 'ansible',
                        colorized: true
                )


                }
            }

            
        }

        stage('Run proxyCommerce front container on remote server'){
            steps{
                    ansiblePlaybook(
                        vaultCredentialsId: 'ansible-vault',
                        inventory: 'ansible/inv',
                        playbook: 'ansible/run_proxy_front_container.yaml',
			installation: 'ansible',
                        colorized: true
                )

            }

        }

        stage('Build spring application'){
            steps{
                script{
                   sh ' mvn clean package'
   		   sh 'echo "build spring application"'
                }
            }
        }

        stage('Build spring application docker image'){
            steps{
                script{
                    sh 'docker build Dockerfile -t proxyCommerce-spring .'
                    sh 'docker image save -o proxyCommerce-spring.tar proxyCommerce-spring'
                    ansiblePlaybook(
                        vaultCredentialsId: 'ansible-vault',
                        inventory: 'ansible/inv',
                        playbook: 'ansible/copy_proxyCommerce_spring_docker_image.yaml',
			installation: 'ansible',
                        colorized: true
                    )
                }
            }
        }

        stage('Run proxyCommerce spring application container on remote server 163'){
            steps{
                    ansiblePlaybook(
                        vaultCredentialsId: 'ansible-vault',
                        inventory: 'ansible/inv',
                        playbook: 'ansible/run_proxyCommerce_spring_container.yaml',
			installation: 'ansible',
                        colorized: true
                )

            }

        }
    }

    post {
       always {
           script {
               def userName = manager.build.getCause(hudson.model.Cause$UserIdCause).getUserName()
               manager.addShortText("Branch: ${BRANCH}","blue", "white", "1px", "grey" )
               manager.addShortText("rev: "+"${env.GIT_REVISION}","black", "white", "0px", "grey")
               manager.addShortText("["+userName+"]","black", "white", "1px", "grey")
               manager.addShortText(" remoteDockerServer ","black", "white", "1px", "grey")
        
           }
       }
    }
}
