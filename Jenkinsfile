podTemplate(label: 'docker',
  containers: [containerTemplate(name: 'docker', image: 'docker:1.11', ttyEnabled: true, command: 'cat')],
  volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]
) {
  node('docker') {
    currentBuild.result = "SUCCESS"
    def IMAGE_NAME = 'avif/docker-helm'
    def BUILD_TAG = "${env.BRANCH_NAME}-build-${env.BUILD_NUMBER}"

    stage('Checking out source control...') {
      checkout scm
    }

    stage('Logging in Docker hub') {
      withCredentials([
        [
          $class          : 'UsernamePasswordMultiBinding',
          credentialsId   : 'docker-hub',
          usernameVariable: 'DOCKER_USERNAME',
          passwordVariable: 'DOCKER_PASSWORD'
        ]
      ]) {
        container('docker') {
          sh "docker login -u ${env.DOCKER_USERNAME} -p ${env.DOCKER_PASSWORD}"
        }
      }
    }

    stage('Testing') {
      env.NODE_ENV = "test"
      print "Environment will be : ${env.NODE_ENV}"
    }

    stage('Building Docker image') {
      container('docker') {
        sh "docker build -t ${IMAGE_NAME}:${env.BRANCH_NAME} ."
      }
    }

    stage('Tagging image') {
      container('docker') {
        print "Tagging image as: ${BUILD_TAG}"
        sh "docker tag ${IMAGE_NAME}:${env.BRANCH_NAME} ${IMAGE_NAME}:${BUILD_TAG}"

        print "Tagging image as: latest"
        sh "docker tag ${IMAGE_NAME}:${env.BRANCH_NAME} ${IMAGE_NAME}:latest"
      }
    }

    stage('Pushing image') {
      container('docker') {
        print "Pushing image as: ${env.BRANCH_NAME}"
        sh "docker push ${IMAGE_NAME}:${env.BRANCH_NAME}"

        print "Pushing image as: ${BUILD_TAG}"
        sh "docker push ${IMAGE_NAME}:${BUILD_TAG}"

        print "Pushing image as: latest"
        sh "docker push ${IMAGE_NAME}:latest"
      }
    }
  }
}
