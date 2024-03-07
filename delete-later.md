```
        
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

```

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

```

```
// ### for loop

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

```

def updateHelmValues(String fileName, String valuesPath) {
    sh "yq eval '.image.tag = \"${env.IMAGE_TAG}\"' -i ${valuesPath}/${fileName}"
}

def gitCommitAndPush(String repoDirectory) {
    try {
        echo "Committing and pushing changes..."

        sh """
        set -x
        git add .
        git commit -m 'Promote environment with new image tag: ${env.IMAGE_TAG}'
        git push origin ${env.HELM_REPO_BRANCH}
        """

        echo "Changes have been pushed successfully."
    } catch (Exception e) {
        echo "Error while committing and pushing: ${e.getMessage()}"
        throw e // Rethrow the exception to handle it at a higher level or fail the build
    }
}

// Then you can call deployToEnvironment like this:
deployToEnvironment('dev', ['euw2', 'use1'])


```

```bash

// Testing

def deployToEnvironment(String environmentPart, String regions) {
    def repoDirectory = "helm-${UUID.randomUUID().toString().take(8)}"
    cloneRepository(repoDirectory)
    
    def regionList = regions.tokenize(',')
    
    dir(repoDirectory) {
        regionList.each { regionPart ->
            def fileNameWeb = "values-${environmentPart}-${regionPart}.yaml"
            def fileNameQueue = "values-${environmentPart}-${regionPart}.yaml"

            // Update Helm values for web and queue for each region
            updateHelmValues(fileNameWeb, env.VALUES_PATH_WEB)
            updateHelmValues(fileNameQueue, env.VALUES_PATH_QUEUE)
        }

        // After all updates are done for each region, commit and push
        gitCommitAndPush(repoDirectory, env.HELM_REPO_BRANCH)
    }
}


//

steps {
    script {
        def regionsList = params.DEPLOY_TO_ALL_REGIONS ? env.ALL_REGIONS.tokenize(',') : [params.REGION]
        ['sbox', 'dev'].each { envName ->
            sshagent(['django-github-k8s']) {
                container('aws') {
                    regionsList.each { region ->
                        echo "Deploying ${envName} to region ${region}"
                        deployToEnvironment(envName, [region]) // Pass region in a list if required
                    }
                }
            }
        }
    }
}


```

```
def deployToEnvironment(String environmentPart, String regions) {
    def repoDirectory = "helm-${UUID.randomUUID().toString().take(8)}"
    try {
        setGitConfig() // Ensuring git configuration is set
        cloneRepository(repoDirectory)

        def regionList = regions.tokenize(',')

        dir(repoDirectory) {
            regionList.each { regionPart ->
                try {
                    def fileNameWeb = "values-${environmentPart}-${regionPart}.yaml"
                    def fileNameQueue = "values-${environmentPart}-${regionPart}.yaml"

                    // Attempt to update Helm values for web and queue for each region
                    updateHelmValues(fileNameWeb, env.VALUES_PATH_WEB)
                    updateHelmValues(fileNameQueue, env.VALUES_PATH_QUEUE)
                } catch (Exception e) {
                    echo "An error occurred while processing region ${regionPart}: ${e.message}"
                    // No throw statement here; log and continue with the next region
                }
            }

            // Attempt to commit and push changes even if there were errors in previous steps
            gitCommitAndPush(repoDirectory, env.HELM_REPO_BRANCH)
        }
    } catch (Exception e) {
        echo "An error occurred during deployment: ${e.message}"
        // No re-throwing the exception here to allow the pipeline to continue
    }
}

```



```bash

// setup

def deployToEnvironment(String environmentPart, String regions) {
    def repoDirectory = "helm-${UUID.randomUUID().toString().take(8)}"
    try {
        setGitConfig() // Ensure git configuration is set before operations
        cloneRepository(repoDirectory)

        def regionList = regions.tokenize(',')

        dir(repoDirectory) {
            regionList.each { regionPart ->
                def fileNameWeb = "values-${environmentPart}-${regionPart}.yaml"
                def fileNameQueue = "values-${environmentPart}-${regionPart}.yaml"

                // Check if the Helm values file exists for web and queue for each region before updating
                if (fileExists("${env.VALUES_PATH_WEB}/${fileNameWeb}")) {
                    updateHelmValues(fileNameWeb, env.VALUES_PATH_WEB)
                } else {
                    echo "File ${env.VALUES_PATH_WEB}/${fileNameWeb} does not exist"
                }
                
                if (fileExists("${env.VALUES_PATH_QUEUE}/${fileNameQueue}")) {
                    updateHelmValues(fileNameQueue, env.VALUES_PATH_QUEUE)
                } else {
                    echo "File ${env.VALUES_PATH_QUEUE}/${fileNameQueue} does not exist"
                }
            }

            // Attempt to commit and push changes even if there were errors in previous steps
            gitCommitAndPush(repoDirectory, env.HELM_REPO_BRANCH)
        }
    } catch (Exception e) {
        echo "An error occurred during deployment: ${e.message}"
        // Errors are logged but do not halt execution, allowing the pipeline to continue
    }
}


```


```
steps {
    script {
        sshagent(['django-github-k8s']) {
            container('aws') {
                def regionsList = params.DEPLOY_TO_ALL_REGIONS ? env.ALL_REGIONS.tokenize(',') : [params.REGION]
                regionsList.each { region ->
                    echo "Deploying to stage in region ${region}"
                    deployToEnvironment('stage', region)
                }
            }
        }
    }
}


```

```bash
def deployToEnvironment(String environmentPart, String regions) {
    def repoDirectory = "helm-${UUID.randomUUID().toString().take(8)}"
    try {
        setGitConfig()
        cloneRepository(repoDirectory)

        def regionList = regions.tokenize(',')

        dir(repoDirectory) {
            regionList.each { regionPart ->
                def fileNameWeb = "values-${environmentPart}-${regionPart}.yaml"
                def fileNameQueue = "values-${environmentPart}-${regionPart}.yaml"

                if (!fileExists("${env.VALUES_PATH_WEB}/${fileNameWeb}")) {
                    echo "WARNING: File ${env.VALUES_PATH_WEB}/${fileNameWeb} does not exist. Skipping update for this file."
                } else {
                    updateHelmValues(fileNameWeb, env.VALIES_PATH_WEB)
                }

                if (!fileExists("${env.VALUES_PATH_QUEUE}/${fileNameQueue}")) {
                    echo "WARNING: File ${env.VALUES_PATH_QUEUE}/${fileNameQueue} does not exist. Skipping update for this file."
                } else {
                    updateHelmValues(fileNameQueue, env.VALIES_PATH_QUEUE)
                }
            }

            gitCommitAndPush(repoDirectory, env.HELM_REPO_BRANCH)
        }
    } catch (Exception e) {
        echo "An error occurred during deployment: ${e.message}. Continuing with the pipeline."
    }
}


```

