pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = 'ac0cbc34-27aa-4c7b-9822-1244a8e33702'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }


    stages {
        
        stage('Build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo 'small change'
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        } 
        stage('Tests'){
            parallel{
                stage('Unit Test') {
                    agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            directory="build"
                            pattern="index.html"

                            find "$directory" -type f | grep "$pattern"

                            npm test
                        '''
                    }
                }

                stage('E2E') {
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }              
        
            } 
        }
        


        stage('Deploying stage') {           

            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment{
                CI_ENVIRONMENT_URL = 'STAGING_URL'
            } 

            steps {
                sh '''
                    npm install netlify-cli node-jq 
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
                    npx playwright test --reporter=html
                '''
            }

            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlaywrightHTML Staging E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }  

        stage('Approval') {
            steps {
                input message: 'Do yoou wish to deploy to production', ok: 'Yes, I am sure!'
            }

        } 
         


        stage('Deploy Prod') {
            environment{
                CI_ENVIRONMENT_URL = 'https://playful-cupcake-c51acb.netlify.app'
            }            

            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    node --version
                    npm install netlify-cli 
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }

            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlaywrightHTML Prod E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }                
  
    }
}
