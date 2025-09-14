pipeline {
  agent any
  environment {
    PATH = "/opt/sonar-scanner/bin:${env.PATH}"
    DOCKER_IMAGE = "sri642/bms-bms:${BUILD_NUMBER}"
    DOCKERHUB_CREDENTIALS = credentials('Docker-token')
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
    stage('Check Java Version') {
       steps {
         sh 'java -version'
         sh 'echo $JAVA_HOME'
      }
    }

    stage('SonarQube Analysis') {
       steps {
         withSonarQubeEnv('SonarQube') {
           sh '''
           export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
           export PATH=$JAVA_HOME/bin:$PATH
          sonar-scanner -Dsonar.projectKey=BookMyShow -Dsonar.sources=.
          '''
      }
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
            def app = docker.build(env.DOCKER_IMAGE)
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
          sh "docker run -d -p 3000:3000 ${DOCKER_IMAGE}"
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
