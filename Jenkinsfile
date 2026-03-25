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
                    
                    // Identify which branch triggered the build
                    def branch = sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
                    if (branch == 'HEAD') {
                        branch = sh(script: "git name-rev --name-only HEAD | sed 's|remotes/origin/||' | sed 's|tags/||'", returnStdout: true).trim()
                    }
                    env.GIT_BRANCH_NAME = branch
                    echo "Working on branch: ${env.GIT_BRANCH_NAME}"
                }
            }
        }

        stage('Push to GitHub') {
            steps {
                sshagent([env.SSH_CREDS]) {
                    script {
                        def currentBranch = env.GIT_BRANCH_NAME
                        
                        // Clean up the local Jenkins state to match GitHub exactly
                        sh "git fetch origin"
                        sh "git checkout -B ${currentBranch} origin/${currentBranch}"

                        // Push only the current branch
                        echo "Finalizing push specifically for ${currentBranch}..."
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
            echo "Push failed. This usually means the branch on GitHub moved while Jenkins was running."
        }
    }
}
