def COLOR_MAP = [
  'SUCCESS' : 'good',
  'FAILURE' : 'danger',
]


pipeline{
  agent any
  tools{
    maven "MAVEN3.9"
    jdk "JDK17"
  }

  environment {
        registryCredential = 'ecr:us-east-1:awscreds'
        appRegistry = "957563772273.dkr.ecr.us-east-1.amazonaws.com/vprofileappimg"
        vprofileRegistry = "https://957563772273.dkr.ecr.us-east-1.amazonaws.com"
        cluster = "vprofile"
        service = "vprofileapptask-service-latest"
    }


  stages{
    stage('Fetch Code') {
      steps{
        git branch: 'docker', url: 'https://github.com/hkhcoder/vprofile-project'
      }
    }


    stage('Build') {
      steps{
        sh 'mvn install -Dskip'
      }
      post{
        success{
          echo "Archiving artifacts"
          archiveArtifacts artifacts: '**/*.war'
        }
      }
    }

    stage('Unit Test') {
      steps{
        sh 'mvn test'
      }
    }

    stage('Checkstyle Analysis') {
      steps{
        sh 'mvn checkstyle:checkstyle'
      }
    }

    stage("Sonar Code Analysis") {
      environment {
        scannerHome = tool 'sonar6.2'
      }
      steps {
        withSonarQubeEnv('sonarserver') {
          sh """${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml"""
        }
      }
    }

    stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
    }

    

    stage('Build App Image') {
          steps {
       
            script {
                dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
                }
          }
    
        }

     stage('Upload App Image') {
          steps{
            script {
              docker.withRegistry( vprofileRegistry, registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
          }
        }

    stage('Remove Container Image') {
        steps{
            sh 'docker rmi -f $(docker images -a -q)'
        }
    }

    stage('Deploy to ecs') {
          steps {
            withAWS(credentials: 'awscreds', region: 'us-east-1') {
            sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
               }
          }
        }

  }

  post{
    always{
      echo 'Slack Notification.'
      slackSend channel: 'help-jenkins',
        color: COLOR_MAP[currentBuild.currentResult],
        message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}" 
    }
  }
}