pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = 'ac0cbc34-27aa-4c7b-9822-1244a8e33702'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"

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

        stage('AWS') {
            agent{
                docker{
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                }
            }
            environment{
                AWS_S3_BUCKET = 'jenkins-20241030'
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version 
                        aws s3 sync build . s3://mAWS_S3_BUCKET             
                    '''
                }
                
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
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            serve -s build &
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
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment{
                CI_ENVIRONMENT_URL = 'STAGING_URL'
            } 

            steps {
                sh '''
                    netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
                    npx playwright test --reporter=html
                '''
            }

            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlaywrightHTML Staging E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }  


         


        stage('Deploy Prod') {
            environment{
                CI_ENVIRONMENT_URL = 'https://playful-cupcake-c51acb.netlify.app'
            }            

            agent{
                docker{
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    node --version
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
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
