pipeline {
    agent any

    environment {
        // ID of your GitHub SSH credentials in Jenkins
        SSH_CREDS = 'github-ssh-key'
        REPO_URL = 'git@github.com:simar-ops/simple-repo-multibranch.git'
    }

    stages {
        stage('Identity & Branch Check') {
            steps {
                script {
                    // 1. Set Git Identity for Simar
                    sh 'git config user.email "simar-ops@users.noreply.github.com"'
                    sh 'git config user.name "Simar"'
                    
                    // 2. Advanced Branch Detection (Fixes "null" or "HEAD" errors)
                    def branch = sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
                    
                    if (branch == 'HEAD') {
                        // If Jenkins is detached, find the real branch name from the remote
                        branch = sh(script: "git name-rev --name-only HEAD | sed 's|remotes/origin/||' | sed 's|tags/||'", returnStdout: true).trim()
                    }
                    
                    env.GIT_BRANCH_NAME = branch
                    echo "Working on branch: ${env.GIT_BRANCH_NAME}"
                }
            }
        }

        stage('Sync & Push') {
            steps {
                sshagent([env.SSH_CREDS]) {
                    script {
                        def currentBranch = env.GIT_BRANCH_NAME
                        
                        // 1. Initial pull to ensure Jenkins workspace is current
                        sh "git pull origin ${currentBranch} --rebase || echo 'No remote changes to pull'"

                        // 2. Automated Sync: If pushing to main, also update dev
                        if (currentBranch == 'main') {
                            echo "Merging main changes into dev branch..."
                            
                            // Switch to dev and update it
                            sh "git checkout dev || git checkout -b dev"
                            sh "git pull origin dev --rebase || echo 'New dev branch'"
                            
                            // Merge and Push dev
                            sh "git merge main --no-edit"
                            sh "git push ${env.REPO_URL} dev"
                            
                            // IMPORTANT: Switch back to main and RE-PULL 
                            // This prevents the 'non-fast-forward' error
                            sh "git checkout main"
                            sh "git pull origin main --rebase"
                        }

                        // 3. Final push of the current branch back to GitHub
                        echo "Finalizing push for ${currentBranch}..."
                        sh "git push ${env.REPO_URL} ${currentBranch}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Successfully synchronized branches for multibranch pipeline project!"
        }
        failure {
            echo "Pipeline failed. Check the logs for Merge Conflicts or Push Rejections."
        }
    }
}
