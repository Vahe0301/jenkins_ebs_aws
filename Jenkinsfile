pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }

    agent any
    environment {
        AWS_REGION = 'us-west-2'
        AWS_ACCOUNT_ID = "263378412066"
        AWS_EB_ENV_NAME = "Jenkinstest-env"
        
    }
    stages {
        stage('Chechkout SCM') {
            steps {
                script {
                    git branch: 'main' ,
                        credentialId: 'github_key' ,
                        url: 'git@github.com:Vahe0301/jenkins_ebs_aws.git'
                }

            }
        }

        stage('Zip Application Code') {
            steps {
                sh 'zip -r my-app.zip .'
            }
        }
        stage ('Upload to S3') {
            steps {
                withCredentials([usernamePassword(credentialsID: 'aws_ebs_key' , accessKeyVariable: "AKIAT2UUTUIRC2FV4AKT" , secretKeyVariable: "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")]) {
                    sh 'aws s3 cp my-app.zip s3:/elasticbeanstalk-us-west-2-263378412066/'
                }
            }
        }
        stage('Deploy to Elastic Beanstalk') {
            steps {
                script {
                    withAWS (credentials: 'aws_ebs_key' , region: env.AWS_RGION) {
                        sh '''
                        aws elasticbeanstalk create-application-version --application-name jenkins_test \
                        --version-label Jenkins-${BUILD_ID} --source-bundle S3Bucket=elasticbeanstalk-us-west-2-263378412066,S3Key=my-app.zip
                        
                        aws elasticbeanstalk update-environment --environment-name $AWS_EB_ENV_NAME --version-label Jenkins-${BUILD_ID}
                        '''
                    }
                }
            }
        }
    }
    }
