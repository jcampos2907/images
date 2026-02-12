pipeline {
  agent {
    kubernetes {
      defaultContainer "jnlp"
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:27-dind
    securityContext:
      privileged: true
    env:
    - name: DOCKER_TLS_CERTDIR
      value: ""
    volumeMounts:
    - name: docker-cache
      mountPath: /var/lib/docker
  volumes:
  - name: docker-cache
    persistentVolumeClaim:
      claimName: docker-cache-pvc
"""
    }
  }

  environment {
    DOCKER_NAMESPACE = "jicamposr"
    VAULT_PATH       = "kv/apps/jenkins"
  }

  stages {

    stage("Detect Changed Image Folders") {
      steps {
        container("jnlp") {
          script {
            env.GIT_SHA = sh(script: "git rev-parse --short=8 HEAD", returnStdout: true).trim()
            def head = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
            def base = ""

            if (env.CHANGE_TARGET) {
              sh "git fetch origin ${env.CHANGE_TARGET}:${env.CHANGE_TARGET} --depth=1 || true"
              base = sh(script: "git rev-parse ${env.CHANGE_TARGET}", returnStdout: true).trim()

            } else if (env.GIT_PREVIOUS_SUCCESSFUL_COMMIT) {
              base = env.GIT_PREVIOUS_SUCCESSFUL_COMMIT

            } else {
              echo "First build detected. Building ALL image folders."
              def allDirs = sh(
                script: "find . -mindepth 2 -maxdepth 2 -name Dockerfile -printf '%h\n' | sed 's|^./||' | cut -d/ -f1 | sort -u",
                returnStdout: true
              ).trim()
              env.CHANGED_DIRS = allDirs ?: ""
              echo "Image dirs for first build: ${env.CHANGED_DIRS}"
              return
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

            def changedInFolders = changedRaw
              .split("\\n")
              .findAll { it.contains("/") }

            if (changedInFolders.isEmpty()) {
              env.CHANGED_DIRS = ""
              echo "Only top-level files changed. No images to build."
              return
            }

            def dirs = changedInFolders
              .collect { it.tokenize('/')[0] }
              .unique()

            def imageDirs = dirs.findAll { d -> fileExists("${d}/Dockerfile") }

            env.CHANGED_DIRS = imageDirs.join(" ")
            echo "Changed image dirs: ${env.CHANGED_DIRS ?: 'none'}"
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
        container("docker") {
          sh '''
            echo "Waiting for Docker daemon..."
            timeout=60
            while [ $timeout -gt 0 ]; do
              if docker info >/dev/null 2>&1; then
                echo "Docker daemon is ready"
                break
              fi
              sleep 2
              timeout=$((timeout - 2))
            done

            if ! docker info >/dev/null 2>&1; then
              echo "ERROR: Docker daemon failed to start"
              exit 1
            fi

            # Register QEMU for cross-platform builds
            echo "Setting up QEMU for multi-arch builds..."
            docker run --rm --privileged tonistiigi/binfmt:latest --install all
            
            echo "Enabled platforms:"
            docker run --rm --privileged tonistiigi/binfmt:latest

            echo "${DOCKERHUB_PASS}" | docker login -u "${DOCKERHUB_USER}" --password-stdin
          '''

          def dirs = env.CHANGED_DIRS?.trim()
            ? env.CHANGED_DIRS.split("\\s+")
            : []

          for (dir in dirs) {
            script {
              def version = readFile("${dir}/version.txt").trim()
              def imageName = dir

              echo "== Building ${imageName}:${version} for linux/arm64 and linux/amd64 =="

              sh """
                set -euo pipefail

                docker build \
                  --platform linux/arm64 \
                  --build-arg VERSION=${version} \
                  --build-arg TARGETARCH=arm64 \
                  --tag ${DOCKER_NAMESPACE}/${imageName}:${version}-arm64 \
                  ${dir}

                docker build \
                  --platform linux/amd64 \
                  --build-arg VERSION=${version} \
                  --build-arg TARGETARCH=amd64 \
                  --tag ${DOCKER_NAMESPACE}/${imageName}:${version}-amd64 \
                  ${dir}

                docker push ${DOCKER_NAMESPACE}/${imageName}:${version}-arm64
                docker push ${DOCKER_NAMESPACE}/${imageName}:${version}-amd64

                # Create multi-arch manifest for version tag
                docker manifest create ${DOCKER_NAMESPACE}/${imageName}:${version} \
                  --amend ${DOCKER_NAMESPACE}/${imageName}:${version}-arm64 \
                  --amend ${DOCKER_NAMESPACE}/${imageName}:${version}-amd64
                docker manifest push ${DOCKER_NAMESPACE}/${imageName}:${version}

                # Create multi-arch manifest for latest tag
                docker manifest create ${DOCKER_NAMESPACE}/${imageName}:latest \
                  --amend ${DOCKER_NAMESPACE}/${imageName}:${version}-arm64 \
                  --amend ${DOCKER_NAMESPACE}/${imageName}:${version}-amd64
                docker manifest push ${DOCKER_NAMESPACE}/${imageName}:latest

                echo "== Done: ${imageName}:${version} (arm64 + amd64) =="
              """
            }
          }

          sh "docker logout"
        }
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