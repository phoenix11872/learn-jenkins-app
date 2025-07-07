pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID='bec995c5-df15-4262-8a7d-665eedfa7399'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        
    }

    stages {
        stage('Cleanup'){
            steps{
                cleanWs()
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
                checkout scm
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
                                image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                                reuseNode true
                                
                            }
                        }
                        steps{
                            sh '''
                                npm i serve
                                node_modules/.bin/serve -s build &
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
        stage('Deploy Staging'){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    npm install netlify-cli@20.1.1 node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > staging-output.json
                    node_modules/.bin/node-jq -r '.deploy_url' staging-output.json

                '''
            }
        }
        stage('Approval'){
            steps{
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }
                
            }
        }
        stage('Deploy Prod'){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
        stage('Prod E2E test')
            {
                agent{
                    docker{
                        image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                        reuseNode true
                        
                    }
                }
                environment{
                    CI_ENVIRONMENT_URL = 'https://storied-dango-dca89c.netlify.app'
                }
                steps{
                    sh '''
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
