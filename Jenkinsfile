pipeline {
    agent { label "Jenkins-Agent" }

    parameters {
        string(
            name: "IMAGE_TAG",
            defaultValue: "",
            description: "Docker image tag received from the CI pipeline"
        )
    }

    environment {
        IMAGE_REPOSITORY = "rashmikaharshamal/register-app"
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: "main",
                    credentialsId: "github",
                    url: "https://github.com/RashmikaHarshamal/gitops-register-app.git"
            }
        }

        stage("Validate Image Tag") {
            steps {
                script {
                    if (!params.IMAGE_TAG?.trim()) {
                        error(
                            "IMAGE_TAG is empty. Run the main register-app CI pipeline."
                        )
                    }

                    echo "Received IMAGE_TAG: ${params.IMAGE_TAG}"
                }
            }
        }

        stage("Update Deployment Image") {
            steps {
                sh """
                    echo "Image before update:"
                    grep "image:" deployment.yaml

                    sed -i \
                      "s|image: ${IMAGE_REPOSITORY}:.*|image: ${IMAGE_REPOSITORY}:${params.IMAGE_TAG}|" \
                      deployment.yaml

                    echo "Image after update:"
                    grep "image:" deployment.yaml
                """
            }
        }

        stage("Commit and Push Deployment") {
            steps {
                script {
                    sh """
                        git config user.name "RashmikaHarshamal"
                        git config user.email "rashmikaharshamal169@gmail.com"
                        git add deployment.yaml
                    """

                    def changeStatus = sh(
                        script: "git diff --cached --quiet",
                        returnStatus: true
                    )

                    if (changeStatus != 0) {
                        sh """
                            git commit -m "Update deployment image to ${params.IMAGE_TAG}"
                        """

                        withCredentials([
                            gitUsernamePassword(
                                credentialsId: "github",
                                gitToolName: "Default"
                            )
                        ]) {
                            sh """
                                git push origin HEAD:main
                            """
                        }
                    } else {
                        echo "Deployment already uses ${params.IMAGE_TAG}. Push skipped."
                    }
                }
            }
        }
    }
}
