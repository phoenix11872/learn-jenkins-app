pipeline {
    agent any
    environment{
        
        REACT_APP_VERSION="1.0.$BUILD_ID"
        AWS_DEFAULT_REGION= 'eu-north-1'
    }

    stages {
        stage('Cleanup'){
            steps{
                cleanWs()
                checkout scm
            }
        }
        
        stage('deploy to AWS'){
            agent{
                docker{
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                
                }
            }
            environment{
                
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws-learning-s3', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh'''
                        aws --version
                        aws ecs register-task-definition --cli-input-json file://aws/task-definition.json
                    '''
                }
                
            }
        }
        stage('Build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            
           
            steps {
                
                sh '''
                    echo 'Small Change'
                    ls -la
                    node --version
                    npm --version
                    
                    npm ci 
                    npm run build
                    ls -la
                    
                '''
            }
        }
           
    }
    
}
