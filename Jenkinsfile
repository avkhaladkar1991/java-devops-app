pipeline {
  agent any

  tools {
    maven 'maven-3'
  }

  environment {
    IMAGE_NAME = "avkhaladkar1991/springboot-gitops-demo"
    IMAGE_TAG  = "${BUILD_NUMBER}"
  }

  stages {

    stage('Checkout') {
      steps {
        git credentialsId: 'github-creds',
            url: 'https://github.com/avkhaladkar1991/java-devops-app.git',
            branch: 'main'
      }
    }

    stage('Build App') {
      steps {
        dir('app') {
          sh 'mvn clean package -DskipTests'
        }
      }
    }

    stage('Docker Build') {
      steps {
        dir('app') {
          sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
        }
      }
    }

    stage('Docker Push') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push ${IMAGE_NAME}:${IMAGE_TAG}
          '''
        }
      }
    }

    stage('Update Helm values') {
      steps {
        sh '''
          sed -i '' 's/tag:.*/tag: '"${IMAGE_TAG}"'/' helm/springboot-app/values.yaml
        '''
      }
    }

    stage('Commit & Push Helm Change') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'github-creds',
          usernameVariable: 'GIT_USER',
          passwordVariable: 'GIT_TOKEN'
        )]) {
          sh '''
            git config user.email "jenkins@ci.com"
            git config user.name "jenkins"
            git commit -am "CI: update image tag ${IMAGE_TAG}" || true
          '''
          // 🔑 Jenkins Git plugin handles the push
          git credentialsId: 'github-creds',
              url: 'https://github.com/avkhaladkar1991/java-devops-app.git',
              branch: 'main'
        }
      }
    }
  }
}
