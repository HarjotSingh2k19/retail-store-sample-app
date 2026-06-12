pipeline {
    // agent any = run this pipeline on any available Jenkins agent
    // since we have only one Jenkins container, it always runs there
    agent any

    environment {
        // DOCKERHUB_USER = your DockerHub username
        // used to tag images as: harjotsingh2k19/retail-store-ui:5
        DOCKERHUB_USER = "harjotsingh2k19"

        // GITOPS_REPO = the second GitHub repo (Helm charts live here)
        // Jenkins clones this in Stage 4 to update the image tag
        GITOPS_REPO    = "https://github.com/HarjotSingh2k19/retail-store-gitops.git"

        // IMAGE_TAG = Jenkins built-in BUILD_NUMBER (auto-increments: 1, 2, 3...)
        // used as Docker image tag so every build produces a unique, traceable image
        // never use "latest" tag in CI/CD — can't rollback to "latest"
        IMAGE_TAG      = "${BUILD_NUMBER}"
    }

    stages {

        // ─────────────────────────────────────────
        // STAGE 1 — CHECKOUT
        // ─────────────────────────────────────────
        stage('1 - Checkout') {
            steps {
                echo "Checking out source code..."

                // checkout scm = checkout the same repo and branch
                // where this Jenkinsfile was found
                // Jenkins already knows the repo URL from job configuration
                // so we don't hardcode it here — no duplication

                // checkout scm is different from git clone.
                // checkout scm doesn't create a new folder. It checks out INTO the existing Jenkins workspace folder directly.

                // git clone → creates a NEW folder (fails if folder already exists ❌)

                // checkout scm → checks out INTO current workspace
                // if repo already exists → just does git pull (updates it) . safe to run multiple times ✅
                checkout scm
            }
        }

        // ─────────────────────────────────────────
        // STAGE 2 — BUILD IMAGES
        // ─────────────────────────────────────────
        stage('2 - Build Images') {
            steps {
                echo "Building all 5 images with tag: ${IMAGE_TAG}"

                // sh = run a shell command on the Jenkins agent
                // docker build -t = tag the image with this name
                // format: dockerhub-username/repo-name:tag
                // src/ui/ = build context (folder containing Dockerfile)
                // we build ALL 5 images before pushing any
                // reason: if a build fails, nothing gets pushed to DockerHub
                // prevents broken images from ever reaching the registry
                sh "docker build -t ${DOCKERHUB_USER}/retail-store-ui:${IMAGE_TAG} src/ui/"
                sh "docker build -t ${DOCKERHUB_USER}/retail-store-catalog:${IMAGE_TAG} src/catalog/"
                sh "docker build -t ${DOCKERHUB_USER}/retail-store-cart:${IMAGE_TAG} src/cart/"
                sh "docker build -t ${DOCKERHUB_USER}/retail-store-orders:${IMAGE_TAG} src/orders/"
                sh "docker build -t ${DOCKERHUB_USER}/retail-store-checkout:${IMAGE_TAG} src/checkout/"
            }
        }

        // ─────────────────────────────────────────
        // STAGE 3 — PUSH TO DOCKERHUB
        // ─────────────────────────────────────────
        stage('3 - Push to DockerHub') {
            steps {
                // withCredentials = fetch credentials from Jenkins credential store
                // injects them as temporary env variables ONLY inside this block
                // outside this block DH_USER and DH_PASS do not exist
                // credentialsId must match the ID we set in Jenkins credentials
                withCredentials([usernamePassword(
                    credentialsId: 'DOCKERHUB_CREDS',
                    usernameVariable: 'DH_USER',   // username stored here
                    passwordVariable: 'DH_PASS'    // password/PAT stored here
                )]) {
                    // echo $DH_PASS | = pipe password to next command via stdin
                    // --password-stdin = read password from stdin NOT from -p flag
                    // why: if you use -p $DH_PASS, password appears in process list
                    // anyone running "ps aux" can see the password — security risk
                    // stdin approach keeps password invisible in process list
                    sh 'echo $DH_PASS | docker login -u $DH_USER --password-stdin'

                    // push all 5 images to DockerHub with the BUILD_NUMBER tag
                    sh "docker push ${DOCKERHUB_USER}/retail-store-ui:${IMAGE_TAG}"
                    sh "docker push ${DOCKERHUB_USER}/retail-store-catalog:${IMAGE_TAG}"
                    sh "docker push ${DOCKERHUB_USER}/retail-store-cart:${IMAGE_TAG}"
                    sh "docker push ${DOCKERHUB_USER}/retail-store-orders:${IMAGE_TAG}"
                    sh "docker push ${DOCKERHUB_USER}/retail-store-checkout:${IMAGE_TAG}"
                }
            }
        }

        // ─────────────────────────────────────────
        // STAGE 4 — UPDATE GITOPS REPO
        // ─────────────────────────────────────────
        stage('4 - Update GitOps Repo') {
            steps {
                // withCredentials with string = fetch a single secret string
                // GitHub PAT is a single token, not username+password
                // so we use Secret text kind, not Username with password
                withCredentials([string(
                    credentialsId: 'GITHUB_TOKEN',
                    variable: 'GH_TOKEN'   // PAT stored in this variable
                )]) {
                    // triple-quoted sh """ """ = multi-line shell script
                    // all commands run in the same shell session
                    sh """
                        # delete gitops folder if it exists from a previous run
                        # without this, git clone fails on Run 2+
                        # error: destination path already exists

                        # checkout scm        → Jenkins manages it  → no cleanup needed
                        # git clone (manual)  → you manage it       → rm -rf before each run
                        
                        rm -rf retail-store-gitops

                        # clone using token authentication
                        # format: https://TOKEN@github.com/user/repo.git
                        # \$GH_TOKEN = backslash escapes $ so shell resolves it
                        # without backslash, Groovy tries to resolve it as
                        # a Groovy variable and fails
                        git clone https://\$GH_TOKEN@github.com/HarjotSingh2k19/retail-store-gitops.git

                        cd retail-store-gitops

                        # sed = stream editor, edits files in place
                        # -i = edit file in place (no backup)
                        # 's|pattern|replacement|g' = substitute globally
                        # pattern:     tag: "anything"  (the .* matches any characters)
                        # replacement: tag: "5"         (current BUILD_NUMBER)
                        # this updates ALL 5 service tags in values.yaml at once
                        # before: tag: "v1"
                        # after:  tag: "5"
                        # ArgoCD detects this change and deploys new images
                        sed -i 's|tag: ".*"|tag: "${IMAGE_TAG}"|g' helm/values.yaml

                        # required before git commit
                        # without these, git refuses to commit saying "who are you?"
                        git config user.email "jenkins@ci.local"
                        git config user.name "Jenkins CI"

                        # stage only values.yaml — only file we changed
                        git add helm/values.yaml

                        # [skip ci] in commit message is CRITICAL
                        # without it: Jenkins sees this push → triggers pipeline again
                        # → updates values.yaml → triggers pipeline → infinite loop
                        # with [skip ci]: GitHub webhook ignores this commit
                        # loop is stopped ✅
                        git commit -m "ci: bump all image tags to ${IMAGE_TAG} [skip ci]"

                        # push updated values.yaml to GitHub
                        # ArgoCD watches this repo and detects the change
                        # within 3 minutes ArgoCD pulls new images and deploys
                        git push
                    """
                }
            }
        }

        // ─────────────────────────────────────────
        // STAGE 5 — DONE
        // ─────────────────────────────────────────
        stage('5 - Done') {
            steps {
                // informational stage — confirms pipeline completed
                // tells developer what happens next (ArgoCD takes over)
                echo "All images pushed with tag: ${IMAGE_TAG}"
                echo "ArgoCD will detect the change and deploy within 3 minutes."
            }
        }
    }

    // ─────────────────────────────────────────────
    // POST — runs after ALL stages finish
    // ─────────────────────────────────────────────
    post {
        // success = only runs if ALL stages passed
        success {
            echo "Pipeline SUCCESS — build ${IMAGE_TAG} deployed via GitOps"
        }

        // failure = only runs if ANY stage failed
        failure {
            echo "Pipeline FAILED — check stage logs above"
        }

        // always = runs regardless of success or failure
        // used for cleanup tasks that must always happen
        always {
            // logout from DockerHub after every run
            // || true = if logout fails for any reason, don't fail the pipeline
            // keeps cleanup safe even if docker daemon is unresponsive
            sh 'docker logout || true'
        }
    }
}
