pipeline {
    agent any
    environment{
        
        REACT_APP_VERSION="1.0.$BUILD_ID"
        AWS_DEFAULT_REGION= 'eu-north-1'
        AWS_ECS_CLUSTER='flawless-tiger-9u4fgn'
        AWS_ECS_SERVICE_PROD='LearnJenkinsApp-TaskDefinition-Prod-service-gsbdv9vi'
        AWS_TD_PROD='LearnJenkinsApp-TaskDefinition-Prod'
    }

    stages {
        stage('Cleanup'){
            steps{
                cleanWs()
                checkout scm
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
        stage('Build Docker image')
        {
            agent{
                docker{
                    image 'my-aws-cli'
                    reuseNode true
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                }
            }
            steps{
                sh'''
                amazon-linux-extras install docker
                docker build -t myjenkinsapp .
                '''
            }
        }
       
        
        stage('deploy to AWS'){
            agent{
                docker{
                    image 'my-aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                
                }
            }
            
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws-learning-s3', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh'''
                        aws --version
                        yum install jq -y
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition.json | jq '.taskDefinition.revision')
                        echo $LATEST_TD_REVISION
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_TD_PROD:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE_PROD
                    '''
                }
                
            }
        }
        
        
           
    }
    
}
