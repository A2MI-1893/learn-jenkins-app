pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '9a4445ef-e51f-42f9-bd29-a9fcb8c26e06'
        NETFLIFY_AUTH_TOKEN = credentials('NETLIFY_TOKEN')
        NETLIFY = 'node_modules/.bin/netlify'
    }

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "A small change"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            test -f build/index.html
                            npm test
                        '''
                    }
                    /*
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    } 
                    */
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test  --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    $NETLIFY --version
                    echo "Deploying to prod. Site ID : $NETLIFY_SITE_ID"
                    $NETLIFY login
                    $NETLIFY status
                    $NETLIFY deploy --dir=build
                '''
            }
        }

        stage ('Approval') {
            steps {
                input message: 'Do you wish to deploy to production ?', ok: 'Yes, I am sure !'
            }
        }

        stage('Deploy prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    $NETLIFY --version
                    echo "Deploying to prod. Site ID : $NETLIFY_SITE_ID"
                    $NETLIFY login
                    $NETLIFY status
                    $NETLIFY deploy --dir=build --prod
                '''
            }
        }

        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://nimble-taffy-74457c.netlify.app'
            }

            steps {
                sh '''
                    npx playwright test  --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
