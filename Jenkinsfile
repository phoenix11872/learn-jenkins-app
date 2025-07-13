pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID='bec995c5-df15-4262-8a7d-665eedfa7399'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION="1.0.$BUILD_ID"
    }

    stages {
        stage('Cleanup'){
            steps{
                cleanWs()
                checkout scm
            }
        }
        stage('AWS'){
            agent{
                docker{
                    image 'amazon/aws-cli'
                }
            }
            steps{
                sh'''
                    aws --version
                '''
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
        stage('Run Tests')
        {
            parallel{
                 stage('Unit Test')
                    {
                        agent{
                            docker{
                                image 'node:18-alpine'
                                reuseNode true
                            }
                        }
                        steps{
                            sh '''
                                echo "Test Stage"
                                test -f build/index.html
                                npm test
                            '''
                        }
                        post{
                            always{
                                junit 'jest-results/junit.xml'
                                
                            }
                        }
                    }
                    stage('E2E test')
                    {
                        agent{
                            docker{
                                image 'myplaywright'
                                reuseNode true
                            }
                        }
                        steps{
                            sh '''
                                serve -s build &
                                sleep 10
                                npx playwright test --reporter=html
                            '''
                        }
                        post{
                            always{
                                
                                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                            }
                        }
                    }
            }
        }
        
        stage('Staging Deploy')
            {
                agent{
                    docker{
                        image 'myplaywright'
                        reuseNode true
                        
                    }
                }
                environment{
                    CI_ENVIRONMENT_URL='STAGIN_URL_TO_BE_SET)'
                }
                
                steps{
                    sh '''
                        
                        netlify --version
                        echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                        netlify status
                        netlify deploy --dir=build --json > staging-output.json
                        CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' staging-output.json)
                        npx playwright test --reporter=html
                    '''
                }
                post{
                    always{
                        
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report Staging', reportTitles: '', useWrapperFileDirectly: true])
                    }
                }
            }
        
        stage('Prod Deploy')
            {
                agent{
                    docker{
                        image 'myplaywright'
                        reuseNode true
                        
                    }
                }
                environment{
                    CI_ENVIRONMENT_URL = 'https://storied-dango-dca89c.netlify.app'
                }
                steps{
                    sh '''
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                    '''
                }
                post{
                    always{
                        
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report Production', reportTitles: '', useWrapperFileDirectly: true])
                    }
                }
            }
       
        
    }
    
}
