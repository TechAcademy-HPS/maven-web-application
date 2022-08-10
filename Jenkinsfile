pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "maven"
    }

    stages {
        stage('checkout') {
            steps {
                git credentialsId: 'SSHKey', url: 'https://github.com/TechAcademy-HPS/maven-web-application.git'
            }
        }
	stage('Build') {
            steps {
                sh "mvn clean package"

            }
        }
        stage('sonar') {
            steps {
                withSonarQubeEnv('SonarQube') {
                sh "mvn sonar:sonar"
                }
               timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
            }

           }
        stage('upload to nexus') {
            steps {
                nexusArtifactUploader artifacts: [
				[
				artifactId: 'maven-web-application',
				classifier: '',
				file: 'target/maven-web-application.war',
				type: 'war'
				]
				],
				credentialsId: 'nexuscredentials',
				groupId: 'com.mt',
				nexusUrl: '13.234.77.182:8081',
				nexusVersion: 'nexus3',
				protocol: 'http',
				repository: 'mavenapp',
				version: '0.0.1'

            }
        }	   
           stage('deploy') {
            steps {
              ansiblePlaybook become: true, disableHostKeyChecking: true, installation: 'Ansible', inventory: 'hosts.inv', playbook: 'tomcatservice.yml'
                 }
            }
			
        }
}
