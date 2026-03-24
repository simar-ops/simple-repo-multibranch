pipeline {
    agent any

    environment {
        // This must match the ID you created in Jenkins Credentials
        SSH_CREDS = 'github-ssh-key' 
        REPO_URL = 'git@github.com:simar-ops/simple-repo-multibranch.git'
    }

    stages {
        stage('Clean & Setup') {
            steps {
                script {
                    // Set your professional DevOps identity
                    sh 'git config user.email "simar-ops@users.noreply.github.com"'
                    sh 'git config user.name "Simar"'
                }
            }
        }

        stage('Sync Branches') {
            steps {
                sshagent([env.SSH_CREDS]) {
                    script {
                        def branches = ['main', 'dev']
                        
                        branches.each { br ->
                            echo "--- Processing Branch: ${br} ---"
                            
                            // 1. Switch to the branch (create if it doesn't exist locally in Jenkins)
                            sh "git checkout ${br} || git checkout -b ${br}"
                            
                            // 2. Pull latest from GitHub to prevent 'Push Rejected' errors
                            // The '|| true' ensures the pipeline doesn't crash if the branch is brand new
                            sh "git pull origin ${br} --rebase || echo 'New branch or nothing to pull'"

                            // 3. If we are on 'dev', bring in the latest changes from 'main'
                            if (br == 'dev') {
                                echo "Merging main into dev..."
                                sh "git merge main --no-edit"
                            }

                            // 4. Push the synchronized branch back to GitHub
                            sh "git push ${env.REPO_URL} ${br}"
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Successfully synchronized main and dev branches!"
        }
        failure {
            echo "Pipeline failed. Check the console output for git conflicts or credential issues."
        }
    }
}
