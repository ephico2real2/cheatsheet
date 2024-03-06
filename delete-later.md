        stage('Build Web Image') {
            when {
                anyOf { branch 'dev'; branch 'prod' }
            }
            steps {
                script {
                    kanikoBuildImage('dev/web/Dockerfile', 'web-portal', "${env.IMAGE_TAG}", "build_type=${env.BUILDTYPE}")
                }
            }
        }

        stage('Build Queue Image') {
            when {
                anyOf { branch 'dev'; branch 'prod' }
            }
            steps {
                script {
                    kanikoBuildImage('dev/queue/Dockerfile', 'queue-portal', "${env.IMAGE_TAG}", "build_type=${env.BUILDTYPE}")
                }
            }
        }


        kanikoBuildImage('dev/web/Dockerfile', 'web-portal', "${env.GIT_COMMIT.take(7)}", "build_type=${env.BUILDTYPE}")

        kanikoBuildImage('dev/web/Dockerfile', 'web-portal', "${env.GIT_COMMIT.take(7)}", "build_type=${env.BUILDTYPE}")



######


def cloneRepository(String repoDirectory) {
    try {
        echo "Cloning repository from ${env.HELM_MANIFEST_REPO} into ${repoDirectory}"
        
        sh """
        set -x
        git clone --single-branch --branch ${env.HELM_REPO_BRANCH} ${env.HELM_MANIFEST_REPO} ${repoDirectory}
        """
        
        echo "Repository cloned successfully into ${repoDirectory}"
    } catch (Exception e) {
        echo "Error while cloning repository: ${e.getMessage()}"
        throw e // Rethrow the exception to handle it at a higher level or fail the build
    }
}

def updateHelmValues(String fileName, String valuesPath) {
    try {
        echo "Updating Helm values file: ${valuesPath}/${fileName} with image tag ${env.IMAGE_TAG}"
        
        sh """
        set -x
        yq eval '.image.tag = \"${env.IMAGE_TAG}\"' -i ${valuesPath}/${fileName}
        """
        
        echo "Helm values file updated successfully."
    } catch (Exception e) {
        echo "Error while updating Helm values file: ${e.getMessage()}"
        throw e // Rethrow the exception to handle it at a higher level or fail the build
    }
}

def gitCommitAndPush(String repoDirectory) {
    dir(repoDirectory) {
        try {
            echo "Committing and pushing changes..."
            sh """
            set -x
            git add .
            git commit -m 'Promote ${env.HELM_REPO_BRANCH} environment with new image tag: ${env.IMAGE_TAG}'
            git push origin ${env.HELM_REPO_BRANCH}
            """
            echo "Changes have been pushed successfully."
        } catch (Exception e) {
            echo "Error while committing and pushing: $e"
            throw e // Re-throw the exception to ensure the build fails appropriately
        }
    }
}




