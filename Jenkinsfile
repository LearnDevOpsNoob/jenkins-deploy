pipeline {
    agent any

    parameters {
        string(name: 'DEPLOY_VERSION', defaultValue: 'v1.0.0', description: 'Version to deploy')
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Git branch to deploy')
        choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: 'Target environment')
    }
    // environment {
    //     // We'll add Netlify related environment variables later.
    //     // For now, you can see how to use parameters in environment if needed.
    //     // Example: NETLIFY_SITE_ID could be chosen based on ENV later.
    //     TEMP_ENV = "placeholder" // just placeholdr as this object cannot be empty
    // }

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
            }
        }
    }

    // Placeholder post section for rollback logic if deployment fails.
    post {
        success {
            echo "✅ Build and Checkout, Deploy successful for ${params.BRANCH_NAME} (version ${params.DEPLOY_VERSION})"
        }
        failure {
            echo "❌ Deployment failed for ${params.BRANCH_NAME}. --- Initiating rollback..."
            // Here you can add rollback logic, e.g., trigger a netlify rollback.
            // For example:
            // sh 'netlify rollback --auth=$NETLIFY_TOKEN --site=$NETLIFY_SITE_ID'
        }
    }
}