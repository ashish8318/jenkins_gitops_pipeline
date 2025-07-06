def gitRepo = ''
def gitUserEmail = ''
def gitUser = ''

pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_URI', defaultValue: '', description: 'Docker image URI to deploy')
    }

    stages {
        stage('Load Env from Credentials') {
            steps {
                withCredentials([
                    string(credentialsId: 'GIT_OPS_REPO', variable: 'GIT_OPS_REPO'),
                    string(credentialsId: 'GIT_AUTHOR_EMAIL', variable: 'AUTHOR_EMAIL'),
                    string(credentialsId: 'GIT_AUTHOR_NAME', variable: 'AUTHOR_NAME')
                ]) {
                    script {
                        gitRepo = GIT_OPS_REPO
                        gitUserEmail = AUTHOR_EMAIL
                        gitUser = AUTHOR_NAME
                    }
                }
            }
        }

        stage("Prepare Workspace") {
            steps {
                script {
                    def repoDir = "gitops-repo"
                    bat "if not exist ${repoDir} mkdir ${repoDir}"
                    dir(repoDir) {
                        bat "git clone ${gitRepo} ."
                    }
                }
            }
        }

        stage('Update Deployment File') {
            steps {
                dir('gitops-repo') {
                    script {
                        echo "📝 Updating deployment.yaml with image: ${params.IMAGE_URI}"

                        def deploymentFile = readFile 'deployment.yaml'
                        def updatedContent = deploymentFile.replaceAll(/image:\s.+/, "image: ${params.IMAGE_URI}")
                        writeFile file: 'deployment.yaml', text: updatedContent

                        // Commit updated file
                        bat """
                        git config user.email "${gitUserEmail}"
                        git config user.name "${gitUser}"
                        git add deployment.yaml
                        git commit -m "Update deployment image to ${params.IMAGE_URI}"
                        """
                    }
                }
            }
        }

        stage("Push to Git Repository") {
            steps {
                dir('gitops-repo') {
                    withCredentials([gitUsernamePassword(credentialsId: 'TOKEN', gitToolName: 'Default')]) {
                        bat "git push origin main"
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment file updated and committed.'
        }
        failure {
            echo '❌ Failed to update deployment file.'
        }
    }
}
