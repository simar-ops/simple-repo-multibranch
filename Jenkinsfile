pipeline {
    agent any

    environment {
        SSH_CREDS = 'github-ssh-key'
        REPO_URL = 'git@github.com:simar-ops/simple-repo-multibranch.git'
    }

    stages {
        stage('Identity & Branch Check') {
            steps {
                script {
                    sh 'git config user.email "simar-ops@users.noreply.github.com"'
                    sh 'git config user.name "Simar"'
                    
                    // This line fixes the 'null' error. 
                    // It looks for the branch name in 3 different places.
                    env.GIT_BRANCH_NAME = env.BRANCH_NAME ?: sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
                    echo "Working on branch: ${env.GIT_BRANCH_NAME}"
                }
            }
        }

        stage('Sync & Push') {
            steps {
                sshagent([env.SSH_CREDS]) {
                    script {
                        def targetBranch = env.GIT_BRANCH_NAME
                        
                        // 1. Pull latest to prevent 'Rejected' errors
                        sh "git pull origin ${targetBranch} --rebase || echo 'New branch'"

                        // 2. If you are on main, also update dev automatically
                        if (targetBranch == 'main') {
                            echo "Updating dev branch from main..."
                            sh "git checkout dev || git checkout -b dev"
                            sh "git merge main --no-edit"
                            sh "git push ${env.REPO_URL} dev"
                            sh "git checkout main" // Switch back to main
                        }

                        // 3. Push the current branch
                        sh "git push ${env.REPO_URL} ${targetBranch}"
                    }
                }
            }
        }
    }
}
