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
    }
    stages {
        stage("Setup") {
            steps {
                script {
                    sh 'echo "$DOCKERHUB_KONGCLOUD_PULL_PSW" | docker login -u "$DOCKERHUB_KONGCLOUD_PULL_USR" --password-stdin || true'
                }
            }
        }
        stage("Bump Version") {
            when {
                branch "main"
                not {
                    // to avoid an infinite loop, we only want to bump the version if
                    // the VERSION file wasn't in the last changeset
                    changeset "HISTORY.md"
                }
            }
            steps {
                script {
                    sh 'docker image rm -f kongcloud/foundation-ci:latest || true'
                    sh 'docker run --env CHANGELOG_GITHUB_TOKEN=$GITHUB_TOKEN --env GITHUB_TOKEN --env GITHUB_USERNAME --env BUILD_NUMBER --volume $(pwd):/workspace kongcloud/foundation-ci:latest bash -c ". /foundation/modules/incl.sh && bump_repo \"/workspace\""'
                }
            }
        }
    }
}
