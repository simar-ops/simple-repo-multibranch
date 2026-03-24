pipeline {
    agent any
    environment {
        SSH_CREDS = 'github-ssh-key' 
    }
    stages {
        stage('Sync Branches') {
            steps {
                sshagent([env.SSH_CREDS]) {
                    script {
                        // Your professional identity
                        sh 'git config user.email "simar-ops@users.noreply.github.com"'
                        sh 'git config user.name "Simar"'
                        
                        def branches = ['main', 'dev']
                        branches.each { br ->
                            echo "Updating branch: ${br}"
                            sh "git checkout ${br} || git checkout -b ${br}"
                            
                            // Merge main into the current branch to keep them identical
                            if (br != 'main') {
                                sh "git merge main --no-edit"
                            }
                            
                            sh "git push git@github.com:simar-ops/simple-repo-multibranch.git ${br}"
                        }
                    }
                }
            }
        }
    }
}
