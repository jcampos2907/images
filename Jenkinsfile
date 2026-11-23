pipeline {
  // Builds run on nacho-3 (Hetzner amd64 VM, Jenkins SSH agent over the tailnet):
  // native amd64 builds with the host Docker, whose layer cache persists between builds.
  agent { label 'nacho-3' }

  options {
    disableConcurrentBuilds()
    buildDiscarder(logRotator(numToKeepStr: '20'))
    timeout(time: 90, unit: 'MINUTES')
  }

  environment {
    DOCKER_NAMESPACE = "jicamposr"
    VAULT_PATH       = "kv/apps/jenkins"
  }

  stages {

    stage("Detect Changed Image Folders") {
      steps {
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
            sh '''#!/bin/bash
                set -euo pipefail
                echo "${DOCKERHUB_PASS}" | docker login -u "${DOCKERHUB_USER}" --password-stdin
              '''

              def dirs = env.CHANGED_DIRS?.trim()
                ? env.CHANGED_DIRS.split("\\s+")
                : []

              for (dir in dirs) {
                script {
                  def version = readFile("${dir}/version.txt").trim()
                  def imageName = dir

                  echo "== Building ${imageName}:${version} =="

                  retry(3) {
                    sh """#!/bin/bash
                      set -euo pipefail

                      docker build \
                        --platform linux/amd64 \
                        --build-arg VERSION=${version} \
                        --build-arg TARGETARCH=amd64 \
                        --tag ${DOCKER_NAMESPACE}/${imageName}:${version} \
                        --tag ${DOCKER_NAMESPACE}/${imageName}:latest \
                        ${dir} || { sleep 15; exit 1; }

                      docker push ${DOCKER_NAMESPACE}/${imageName}:${version}
                      docker push ${DOCKER_NAMESPACE}/${imageName}:latest

                      echo "== Done: ${imageName}:${version} =="
                    """
                  }
                }
              }

              sh "docker logout"
          }
        }
      }
    }
  }

  post {
    always {
      sh 'docker logout >/dev/null 2>&1 || true'
      // Keep a week of build cache and drop dangling images so nacho-3's disk doesn't fill.
      sh 'docker builder prune -f --filter until=168h >/dev/null 2>&1 || true; docker image prune -f >/dev/null 2>&1 || true'
      echo "Done. Built: ${env.CHANGED_DIRS ?: 'none'}"
    }
  }
}