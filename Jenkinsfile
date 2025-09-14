pipeline {
  agent any
  tools {
    jdk 'jdk17'
    nodejs 'node24'
  }
  environment {
    DOCKER_IMAGE = "sri642/bms-bms:${BUILD_NUMBER}"
    DOCKERHUB_CREDENTIALS = credentials('Docker-token')
    SCANNER_HOME = tool 'sonar-scanner'
  }
  stages {
    stage('Clean Workspace') {
      steps {
        cleanWs()
      }
    }
    stage('Checkout Code') {
      steps {
        git branch: 'feature/docker-integration', url: 'https://github.com/Srilatha7525/Book-My-Show-Devops.git'
      }
    }
    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('sonar-server') {
        sh ''' 
          $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BOOK-MY-SHOW \
          -Dsonar.projectKey=Book-my-show
        '''
      }
    }
    stage('Quality Gate') {
      steps {
        script {
          waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube-secret'
        }
      }
    }
    stage('Install Dependencies') {
      steps {
        sh '''
          cd bookmyshow-app
          ls -la  # Verify package.json exists
          if [ -f package.json ]; then
              rm -rf node_modules package-lock.json  # Remove old dependencies
              npm install  # Install fresh dependencies
          else
              echo "Error: package.json not found in bookmyshow-app!"
              exit 1
          fi
        '''
      }
    }
    stage('Docker Build & Push') {
      steps {
        script {
          docker.withRegistry('', env.DOCKERHUB_CREDENTIALS) {
            echo "Building Docker image..."
            def app = docker.build("sri642/bms:latest", "-f bookmyshow-app/Dockerfile bookmyshow-app")
            echo "Pushing Docker image to registry..."
            app.push()
          }
        }
      }
    }
    stage('Deploy to Docker') {
      steps {
        script {
          sh '''
            cid=$(docker ps -q -f "publish=3000")
            if [ ! -z "$cid" ]; then
              docker rm -f $cid
            fi
          '''
          sh "docker run -d -p 3000:3000 sri642/bms:latest"
        }
      }
    }
  }
  post {
    success {
      mail to: 'srilathamaddasani05@gmail.com',
           subject: "Build SUCCESS",
           body: "Jenkins build ${env.BUILD_NUMBER} succeeded! Check Jenkins for details."
    }
    failure {
      mail to: 'srilathamaddasani05@gmail.com',
           subject: "Build FAILED",
           body: "Jenkins build ${env.BUILD_NUMBER} failed. Check Jenkins for details."
    }
  }
}
