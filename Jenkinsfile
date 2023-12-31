pipeline {
    agent any
    environment {
	TAG = "v0.${env.BUILD_NUMBER}"
	S3_BUCKET = "mybucket-us-east-2"
	ARTIFACTS_DIR = '/var/lib/jenkins/workspace/jenkins/src/main/webapp'
    }
    
    stages {
         stage('Compile-Package') {
            steps {
                script {
                    def mvnHome = tool name: 'maven3', type: 'maven'
                    sh "${mvnHome}/bin/mvn clean package"
                    sh 'mv target/myweb*.war target/newapp-${TAG}.war'
                }
            }
        }
	    
    stage('Upload to S3') {
      steps {
	sh '''
          AWS_ACCESS_KEY_ID='aws-credentials'
          AWS_SECRET_ACCESS_KEY='aws-credentials'
          AWS_DEFAULT_REGION=us-east-2
          aws s3 cp target/newapp-${TAG}.war s3://${S3_BUCKET}/newapp-${TAG}.war
            '''
      	}
      }

	stage('SonarQube Analysis') {
            steps {
                script {
                    def mvnHome = tool name: 'maven3', type: 'maven'
                    withSonarQubeEnv('sonar') {
                        sh "${mvnHome}/bin/mvn sonar:sonar"
                    }
                }
            }
        }
	
	stage('Ansible Deployment') {
            steps {
               script {
                  ansiblePlaybook credentialsId: 'private-key', disableHostKeyChecking: true, installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy.yml', extraVars: [
                        's3_bucket': S3_BUCKET,
                        's3_object_key': "newapp-${TAG}.war"
                    ]
                }
            }
        }

       stage('Publish to S3') {
            steps {
                echo 'Publishing artifacts to S3...'
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                    sh "aws s3 cp ${env.ARTIFACTS_DIR} s3://${env.S3_BUCKET}/"
                }
            }
       }
    }
}
