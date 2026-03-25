pipeline {
    agent any

    environment {
        SSH_CREDS = 'github-ssh-key'
        REPO_URL = 'git@github.com:simar-ops/simple-repo-multibranch.git'
    }

    stages {
        stage('Identity Setup') {
            steps {
                script {
                    sh 'git config user.email "simar-ops@users.noreply.github.com"'
                    sh 'git config user.name "Simar"'
                }
            }
        }

        stage('Push to Current Branch') {
            steps {
                sshagent([env.SSH_CREDS]) {
                    script {
                        // env.BRANCH_NAME is a built-in Jenkins variable 
                        // that knows if it is 'main' or 'dev'
                        def currentBranch = env.BRANCH_NAME
                        echo "Detected branch: ${currentBranch}"

                        // 1. Ensure we are on the correct branch locally in Jenkins
                        sh "git checkout ${currentBranch} || git checkout -b ${currentBranch}"

                        // 2. Pull latest to avoid conflicts
                        sh "git pull origin ${currentBranch} --rebase || echo 'Fresh branch'"

                        // 3. Push only the current branch back to GitHub
                        echo "Pushing changes to origin/${currentBranch}..."
                        sh "git push ${env.REPO_URL} ${currentBranch}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Successfully pushed to ${env.BRANCH_NAME}!"
        }
    }
}
