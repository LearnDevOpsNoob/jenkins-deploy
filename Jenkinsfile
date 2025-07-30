pipeline {
    agent any

    parameters {
        string(name: 'DEPLOY_VERSION', defaultValue: 'v1.0.0', description: 'Version to deploy')
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Git branch to deploy')
        choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: 'Target environment')
    }
    environment {
        // We'll add Netlify related environment variables later.
        // For now, you can see how to use parameters in environment if needed.
        // Example: NETLIFY_SITE_ID could be chosen based on ENV later.
        NETLIFY_AUTH_TOKEN = credentials('NETLIFY_AUTH_TOKEN')
    }

    tools {
        nodejs 'Node 22'
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out branch: ${params.BRANCH_NAME}"
                git branch: "${params.BRANCH_NAME}", url: 'https://github.com/LearnDevOpsNoob/jenkins-deploy.git'
            }
        }
        stage('Build') {
            steps {
                // Here you would typically install dependencies and build your Angular app.
                // For example:
                echo "Installing dependencies..."
                bat 'npm install'
                echo "Building the Angular application..."
                bat 'npm run build'
            }
        }
        stage('Deploy to Netlify') {
            steps {
                // We'll populate this stage with Netlify CLI commands later.
                echo "Deploying version ${params.DEPLOY_VERSION} to ${params.ENV} environment from branch ${params.BRANCH_NAME}"
                
                // Example command (for now just echo; later replace with netlify deploy command):
                // sh 'netlify deploy --prod --auth=$NETLIFY_TOKEN --site=$NETLIFY_SITE_ID --dir=dist/your-angular-app/browser'
                
                script {
                    // infer angular app
                    def angularConfig = readJSON file: 'angular.json'
                    def appName = angularConfig['defaultProject']
                    def outputDir = "dist/${appName}/browser"

                    // Deploy using Netlify CLI securely
                    withCredentials([
                        string(credentialsId: 'NETLIFY_AUTH_TOKEN', variable: 'NETLIFY_AUTH_TOKEN'),
                        string(credentialsId: 'NETLIFY_SITE_ID_DEV', variable: 'NETLIFY_SITE_ID_DEV'),
                        string(credentialsId: 'NETLIFY_SITE_ID_STAGING', variable: 'NETLIFY_SITE_ID_STAGING'),
                        string(credentialsId: 'NETLIFY_SITE_ID_PROD', variable: 'NETLIFY_SITE_ID_PROD')
                    ]) {

                    //Site ID mapped to ENV
                    def siteIdMap = [
                        dev: NETLIFY_SITE_ID_DEV,
                        staging: NETLIFY_SITE_ID_STAGING,
                        prod: NETLIFY_SITE_ID_PROD
                    ]

                    def siteId = siteIdMap[params.ENV]
                    echo "Deploying to Netlify site ID: ${siteId} (env: ${params.ENV})"
                    

                    // Deploy using Netlify CLI
                    // withEnv(["NETLIFY_AUTH_TOKEN=${NETLIFY_AUTH_TOKEN}"]) {
                    //     bat """
                    //         netlify deploy --dir=${outputDir} --site=${siteId} --auth=%NETLIFY_AUTH_TOKEN% --prod
                    //     """
                    // }

                        withEnv([
                            "OUTPUT_DIR=${outputDir}",
                            "SITE_ID=${siteId}",
                            "NETLIFY_AUTH_TOKEN=${NETLIFY_AUTH_TOKEN}"
                        ]) {
                            bat 'netlify deploy --dir=%OUTPUT_DIR% --site=%SITE_ID% --auth=%NETLIFY_AUTH_TOKEN% --prod'
                        }
                    }
                    
                }
            }
        }
    }

    // Placeholder post section for rollback logic if deployment fails.
    post {
        success {
            echo "✅ Build and Checkout, Deploy successful for ${params.BRANCH_NAME} (version ${params.DEPLOY_VERSION})"
        }
        failure {
            script {
                echo "❌ Deployment failed for ${params.BRANCH_NAME}. --- Initiating rollback..."

                // Properly bind credentials first
                withCredentials([
                    string(credentialsId: 'NETLIFY_SITE_ID_DEV', variable: 'SITE_ID_DEV'),
                    string(credentialsId: 'NETLIFY_SITE_ID_STAGING', variable: 'SITE_ID_STAGING'),
                    string(credentialsId: 'NETLIFY_SITE_ID_PROD', variable: 'SITE_ID_PROD'),
                    string(credentialsId: 'NETLIFY_AUTH_TOKEN', variable: 'NETLIFY_AUTH_TOKEN')
                ]) {
                    def siteIdMap = [
                        dev: SITE_ID_DEV,
                        staging: SITE_ID_STAGING,
                        prod: SITE_ID_PROD
                    ]
                    def siteId = siteIdMap[params.ENV]

                    // Inject as environment variables
                    withEnv([
                        "NETLIFY_AUTH_TOKEN=${NETLIFY_AUTH_TOKEN}",
                        "SITE_ID=${siteId}"
                    ]) {
                        // ✅ Use %SITE_ID% and %NETLIFY_AUTH_TOKEN% in bat command
                        bat 'netlify rollback --site-id=%SITE_ID% --auth=%NETLIFY_AUTH_TOKEN%'
                    }
                }
            }
    }

    }
}