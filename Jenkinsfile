pipeline {
  agent {
    kubernetes {
      defaultContainer "jnlp"
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:v1.23.2-debug
    command: ["/busybox/cat"]
    tty: true
    volumeMounts:
      - name: docker-config
        mountPath: /kaniko/.docker

  volumes:
    - name: docker-config
      emptyDir: {}
"""
    }
  }

  environment {
    DOCKER_NAMESPACE = "jicamposr"
    CACHE_REPO_ROOT  = "jicamposr/images-cache"
    VAULT_PATH       = "kv/apps/jenkins"
  }

  stages {

    stage("Detect Changed Image Folders") {
      steps {
        // run diff where git + repo already exist
        container("jnlp") {
          script {
            env.GIT_SHA = sh(script: "git rev-parse --short=8 HEAD", returnStdout: true).trim()
            def head = sh(script: "git rev-parse HEAD", returnStdout: true).trim()

            def base = ""
            if (env.CHANGE_TARGET) {
              // PR build: compare against target branch
              sh "git fetch origin ${env.CHANGE_TARGET}:${env.CHANGE_TARGET} --depth=1"
              base = sh(script: "git rev-parse ${env.CHANGE_TARGET}", returnStdout: true).trim()
            } else if (env.GIT_PREVIOUS_SUCCESSFUL_COMMIT) {
              base = env.GIT_PREVIOUS_SUCCESSFUL_COMMIT
            } else {
              base = sh(script: "git rev-parse HEAD~1", returnStdout: true).trim()
            }

            echo "HEAD = ${head}"
            echo "BASE = ${base}"

            def changedRaw = sh(
              script: "git diff --name-only ${base}..${head}",
              returnStdout: true
            ).trim()

            if (!changedRaw) {
              env.CHANGED_DIRS = ""
              echo "No changes detected."
              return
            }

            echo "Changed files:\n${changedRaw}"

            // 🔒 Ignore top-level files explicitly:
            // only keep changes that are in subfolders (contain '/')
            def changedInFolders = changedRaw
              .split("\\n")
              .findAll { it.contains("/") }

            if (changedInFolders.isEmpty()) {
              env.CHANGED_DIRS = ""
              echo "Only top-level files changed. No images to build."
              return
            }

            // top-level dir of each changed file
            def dirs = changedInFolders
              .collect { it.tokenize('/')[0] }
              .unique()

            // only dirs that contain a Dockerfile
            def imageDirs = dirs.findAll { d ->
              fileExists("${d}/Dockerfile")
            }

            if (imageDirs.isEmpty()) {
              env.CHANGED_DIRS = ""
              echo "No image folders changed."
              return
            }

            env.CHANGED_DIRS = imageDirs.join(" ")
            echo "Changed image dirs: ${env.CHANGED_DIRS}"
          }
        }
      }
    }

    stage("Build & Push Changed Images") {
      when { expression { return env.CHANGED_DIRS?.trim() } }
      steps {
        script {
          def secrets = [[
            path: env.VAULT_PATH,
            engineVersion: 2,
            secretValues: [
              [envVar: "DOCKERHUB_USER", vaultKey: "DOCKERHUB_USER"],
              [envVar: "DOCKERHUB_PASS", vaultKey: "DOCKERHUB_PASS"]
            ]
          ]]

          withVault(vaultSecrets: secrets) {

            // write auth once
            container("kaniko") {
              sh """
                set -euo pipefail
                cat > /kaniko/.docker/config.json <<EOF
                {
                  "auths": {
                    "https://index.docker.io/v1/": {
                      "username": "\$DOCKERHUB_USER",
                      "password": "\$DOCKERHUB_PASS"
                    }
                  }
                }
EOF
              """
            }

            def builds = [:]
            def dirs = env.CHANGED_DIRS.split("\\s+")

            dirs.each { dir ->
              builds[dir] = {
                container("kaniko") {
                  script {
                    def version = readFile("${dir}/version.txt").trim()
                    def imageName = dir

                    echo "== Building ${imageName}:${version} =="

                    sh """
                      set -euo pipefail

                      /kaniko/executor \
                        --context ${dir} \
                        --dockerfile ${dir}/Dockerfile \
                        --destination ${DOCKER_NAMESPACE}/${imageName}:${version} \
                        --destination ${DOCKER_NAMESPACE}/${imageName}:${GIT_SHA} \
                        --destination ${DOCKER_NAMESPACE}/${imageName}:latest \
                        --cache=true \
                        --cache-repo ${CACHE_REPO_ROOT}/${imageName} \
                        --snapshot-mode=redo \
                        --use-new-run \
                        --cache-copy-layers \
                        --cache-run-layers
                    """
                  }
                }
              }
            }

            parallel builds
          }
        }
      }
    }
  }

  post {
    always {
      echo "Done. Built: ${env.CHANGED_DIRS ?: 'none'}"
    }
  }
}
