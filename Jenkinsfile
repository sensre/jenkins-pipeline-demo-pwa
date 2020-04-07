pipeline {
  agent {
    docker {
      image 'jenkinsslave:latest'
      registryUrl 'http://8598567586.dkr.ecr.us-west-2.amazonaws.com'
      registryCredentialsId 'ecr:us-east-1:3435443545-5546566-567765-3225'
      args '-v /home/centos/.ivy2:/home/jenkins/.ivy2:rw -v jenkins_opt:/usr/local/bin/opt -v jenkins_apijenkins:/home/jenkins/config -v jenkins_logs:/var/logs -v jenkins_awsconfig:/home/jenkins/.aws --privileged=true -u jenkins:jenkins'
    }

  }
  stages {
    stage('Initialize') {
      steps {
        script {
          notifyBuild('STARTED')
          echo "${BUILD_NUMBER} - ${env.BUILD_ID} on ${env.JENKINS_URL}"
          echo "Branch Specifier :: ${params.SPECIFIER}"
          echo "Deploy to QA? :: ${params.DEPLOY_QA}"
          echo "Deploy to UAT? :: ${params.DEPLOY_UAT}"
          echo "Deploy to PROD? :: ${params.DEPLOY_PROD}"
          sh 'rm -rf target/universal/*.zip'
        }

      }
    }
    stage('Checkout') {
      steps {
        echo 'Checkout Repo'
        git(branch: "${params.SPECIFIER}", url: "${GIT_URL}")
      }
    }
    stage('Build') {
      steps {
        sh 'npm install'
        sh 'ng lint'
        sh 'ng build --prod'
      }
    }
    stage('Static Code Coverage Analysis') {
      parallel {
        stage('Execute Whitesource Analysis') {
          steps {
            whitesource(jobCheckPolicies: 'global', jobForceUpdate: 'global', product: "$WS_PRODUCT_TOKEN", projectToken: "$WS_PROJECT_TOKEN")
          }
        }
        stage('SonarQube analysis') {
          steps {
            sh '/usr/bin/sonar-scanner'
          }
        }
      }
    }
    stage('Docker Tag & Push') {
      steps {
        script {
          branchName = getCurrentBranch()
          shortCommitHash = getShortCommitHash()
          IMAGE_VERSION = "${BUILD_NUMBER}-" + branchName + "-" + shortCommitHash
          sh 'eval $(aws ecr get-login --no-include-email --region us-east-1)'
          sh "docker-compose build"
          sh "docker tag ${REPOURL}/${APP_NAME}:latest ${REPOURL}/${APP_NAME}:${IMAGE_VERSION}"
          sh "docker push ${REPOURL}/${APP_NAME}:${IMAGE_VERSION}"
          sh "docker push ${REPOURL}/${APP_NAME}:latest"

          sh "docker rmi ${REPOURL}/${APP_NAME}:${IMAGE_VERSION} ${REPOURL}/${APP_NAME}:latest"
        }

      }
    }
    stage('Deploy - CI') {
      steps {
        echo 'Deploying to CI Environment.'
      }
    }
    stage('Deploy - QA') {
      when {
        expression {
          params.DEPLOY_QA == true
        }

      }
      steps {
        echo 'Deploy to QA...'
      }
    }
    stage('Deploy - UAT') {
      when {
        expression {
          params.DEPLOY_UAT == true
        }

      }
      steps {
        echo 'Deploy to UAT...'
      }
    }
    stage('Deploy - Production') {
      when {
        expression {
          params.DEPLOY_PROD == true
        }

      }
      steps {
        echo 'Deploy to PROD...'
      }
    }
  }
  environment {
    APP_NAME = 'jenkins-pipeline-demo-pwa'
    BUILD_NUMBER = "${env.BUILD_NUMBER}"
    IMAGE_VERSION = "v_${BUILD_NUMBER}"
    GIT_URL = "git@github.yourdomain.com:mpatel/${APP_NAME}.git"
    GIT_CRED_ID = 'izleka2IGSTDK+MiYOG3b3lZU9nYxhiJOrxhlaJ1gAA='
    REPOURL = 'cL5nSDa+49M.dkr.ecr.us-east-1.amazonaws.com'
    SBT_OPTS = '-Xmx1024m -Xms512m'
    JAVA_OPTS = '-Xmx1024m -Xms512m'
    WS_PRODUCT_TOKEN = 'FJbep9fKLeJa/Cwh7IJbL0lPfdYg7q4zxvALAxWPLnc='
    WS_PROJECT_TOKEN = 'zwzxtyeBntxX4ixHD1iE2dOr4DVFHPp7D0Czn84DEF4='
    HIPCHAT_TOKEN = 'SpVaURsSTcWaHKulZ6L4L+sjKxhGXCkjSbcqzL42ziU='
    HIPCHAT_ROOM = 'NotificationRoomName'
  }
  post {
    always {
      echo 'I AM ALWAYS first'
      notifyBuild "${currentBuild.currentResult}"

    }

    aborted {
      echo 'BUILD ABORTED'

    }

    success {
      echo 'BUILD SUCCESS'
      echo 'Keep Current Build If branch is master'

    }

    unstable {
      echo 'BUILD UNSTABLE'

    }

    failure {
      echo 'BUILD FAILURE'

    }

  }
  options {
    buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '10', numToKeepStr: '20'))
    timestamps()
    timeout(time: 10, unit: 'MINUTES')
  }
  parameters {
    string(defaultValue: 'develop', description: 'Branch Specifier', name: 'SPECIFIER')
    booleanParam(defaultValue: false, description: 'Deploy to QA Environment ?', name: 'DEPLOY_QA')
    booleanParam(defaultValue: false, description: 'Deploy to UAT Environment ?', name: 'DEPLOY_UAT')
    booleanParam(defaultValue: false, description: 'Deploy to PROD Environment ?', name: 'DEPLOY_PROD')
  }
}