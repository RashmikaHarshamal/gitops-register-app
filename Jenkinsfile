pipeline {
    agent { label "Jenkins-Agent" }

    parameters {
        string(
            name: "IMAGE_TAG",
            defaultValue: "",
            description: "Docker image tag, for example: 1.0.0-17"
        )
    }

    environment {
        IMAGE_NAME = "rashmikaharshamal/register-app"
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
                        error("IMAGE_TAG is empty. Provide a tag such as 1.0.0-17.")
                    }
                }
            }
        }

        stage("Update Deployment Image") {
            steps {
                sh """
                    echo "Current deployment:"
                    grep "image:" deployment.yaml

                    sed -i "s|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" deployment.yaml

                    echo "Updated deployment:"
                    grep "image:" deployment.yaml
                """
            }
        }

        stage("Commit and Push Changes") {
            steps {
                sh """
                    git config user.name "RashmikaHarshamal"
                    git config user.email "rashmikaharshamal169@gmail.com"

                    git add deployment.yaml

                    if git diff --cached --quiet; then
                        echo "No deployment change was required."
                    else
                        git commit -m "Update deployment image to ${IMAGE_TAG}"
                    fi
                """

                withCredentials([
                    gitUsernamePassword(
                        credentialsId: "github",
                        gitToolName: "Default"
                    )
                ]) {
                    sh """
                        if git log -1 --pretty=%B | grep -q "Update deployment image to ${IMAGE_TAG}"; then
                            git push origin main
                        else
                            echo "Nothing new to push."
                        fi
                    """
                }
            }
        }
    }
}
