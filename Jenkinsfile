def podlabel = "jenkins-${UUID.randomUUID().toString()}"

pipeline {
  environment {
    IMAGE_TAG = "${sh(script:'echo \$(date +%Y-%m%d-%H%M)', returnStdout: true)}"
    ECR_IMAGE_NAME = "mobiletechnologies/sonarqube"
    ECR_IMAGE_FULLNAME = "$ECR_IMAGE_NAME:$IMAGE_TAG"

    PRODUCT_REPOSITORY = "https://github.com/sbernardello/sonarqube_image.git/"
    PRODUCT_REPOSITORY_BRANCH = "*/main"
}
  agent {
    kubernetes {
        label podlabel
        yaml """
kind: Pod
metadata:
  name: jenkins-slave
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: docker-registry-config
        mountPath: /kaniko/.docker
  restartPolicy: Never
  volumes:
    - name: docker-registry-config
      secret:
        secretName: docker-registry-config
        items:
        - key: .dockerconfigjson
          path: config.json
"""
    }
}
    stages {
      stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                          branches: [[name: env.PRODUCT_REPOSITORY_BRANCH]],
                          doGenerateSubmoduleConfigurations: false,
                          extensions: [],
                          submoduleCfg: [],
                          userRemoteConfigs: [
                              [
                                  credentialsId: 'jenkins-ganimede-github-oauth-token',
                                  url: env.PRODUCT_REPOSITORY
                              ]
                          ]
                        ])
            }
        }
      stage('Build') {
        environment {
            PATH = "/busybox:/kaniko:$PATH"
            // GITHUB_ACCESS_TOKEN = credentials('jenkins-ganimede-github-oauth-token')
        }
        steps {

            container(name: 'kaniko', shell: '/busybox/sh') {

                sh """#!/busybox/sh -xe
                set +x
                echo "\n *** Building sonarqube image *** \n"

                /kaniko/executor -c `pwd` \
                -f `pwd`/Dockerfile.arm64 \
                --cleanup \
                --destination mobiletechnologies/sonarqube:arm64

            """
            }
        }

    }

  }
}
