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
