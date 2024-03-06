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

```
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

### for loop


If you want to ensure that the gitCommitAndPush function is called only after all the values.yaml files for each region and environment have been updated, you can structure your deployToEnvironment function in such a way that it iterates over all the regions, updates the files for each, and only after all updates are done, proceeds to commit and push the changes.

Here's an example of how you could structure this:

groovy
Copy code
def deployToEnvironment(String environmentPart, String[] regions) {
    def repoDirectory = "helm-${UUID.randomUUID().toString().take(8)}"
    cloneRepository(repoDirectory)

    dir(repoDirectory) {
        regions.each { regionPart ->
            def fileNameWeb = "values-${environmentPart}-${regionPart}.yaml"
            def fileNameQueue = "values-${environmentPart}-${regionPart}.yaml"

            // Update Helm values for web and queue for each region
            updateHelmValues(fileNameWeb, env.VALUES_PATH_WEB)
            updateHelmValues(fileNameQueue, env.VALUES_PATH_QUEUE)
        }

        // After all updates are done for each region, commit and push
        gitCommitAndPush(repoDirectory)
    }
}

```




