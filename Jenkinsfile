pipeline{
    agent {
        node {
            label 'hybrid'
        }
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
    }
    environment {
        GITHUB_TOKEN = credentials('BUMPER_GITHUB_TOKEN')
        DOCKERHUB_KONGCLOUD_PULL = credentials('DOCKERHUB_KONGCLOUD_PULL')
        KONG_VERSION = "2.4.0.8"
        KONG_EE_REPO_NAME = "bumper-ee"
    }
    stages {
        stage("Setup") {
            steps {
                script {
                    sh 'echo "$DOCKERHUB_KONGCLOUD_PULL_PSW" | docker login -u "$DOCKERHUB_KONGCLOUD_PULL_USR" --password-stdin || true'
                }
            }
        }
        stage("Bump History") {
            when {
                branch "next/2.4.x.x"
                not {
                    anyOf {
                        // to avoid an infinite loop, we only want to bump the version if
                        // the associated version files were not in the last changeset
                        changeset "*.rockspec"
                        changeset "**/**/meta.lua"
                        changeset "HISTORY.md"
                    }
                }
            }
            steps {
                script {
                    sh 'docker image rm -f kongcloud/foundation-ci:latest || true'
                    sh 'docker run --env KONG_EE_REPO_NAME --env CHANGELOG_GITHUB_TOKEN=$GITHUB_TOKEN --env GITHUB_TOKEN --env GITHUB_USERNAME --env BUILD_NUMBER --volume $(pwd):/workspace kongcloud/foundation-ci:latest bash -c ". /foundation/modules/incl.sh && bump_repo \"/workspace\""'
                }
            }
        }
    }
}
