pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    agent any
    environment {
        AWS_REGION = 'us-west-2'  // Set your AWS region
        AWS_ACCOUNT_ID = "263378412066"
        AWS_EB_ENV_NAME = 'Jenkins-test-env'  // Set your Elastic Beanstalk environment name
    }
    stages {
        stage('Checkout SCM') {
            steps { 
                script {
                    git branch: 'main',
                        credentialsId: 'github_key',
                        url: 'git@github.com:Vahe0301/jenkins_ebs_aws.git'
                }
            }

        }
        // stage('Assume Role') {
        //     steps {
        //         script {
        //             // Assume an AWS IAM role for this pipeline
        //             def assumedRole = awsRoleAssumer(
        //                 credentials: 'aws_eb_access',
        //                 roleArn: 'arn:aws:iam::908177614064:role/aws_eb_roles_for_jenkins'
        //             )
        //         }
        //     }
        stage('Zip Application Code') {
            steps {
                // Zip your application code
                sh "zip -r my-app.zip ."
            }
        }
        stage('Upload to S3') {
            steps {
                // Upload the zipped code to an S3 bucket
                withCredentials([usernamePassword(credentialsId: 'aws_ebs_key', accessKeyVariable: 'AWS_ACCESS_KEY', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh 'aws s3 cp my-app.zip s3:/elasticbeanstalk-us-west-2-263378412066/'
                }
            }
        }
        stage('Deploy to Elastic Beanstalk') {
            steps {
                script {
                    withAWS(credentials: 'aws_ebs_key', region: env.AWS_REGION) {
                        sh '''
                            aws elasticbeanstalk create-application-version --application-name jenkins-test \
                            --version-label Jenkins-${BUILD_ID} --source-bundle S3Bucket=elasticbeanstalk-us-west-2-263378412066,S3Key=my-app.zip

                            aws elasticbeanstalk update-environment --environment-name $AWS_EB_ENV_NAME --version-label Jenkins-${BUILD_ID}
                        '''
                    }
                }
            }
        }
    }
}
        
