def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any

    environment {
        WORKSPACE = "${env.WORKSPACE}"
    }

    tools {
        maven 'localMaven'
        jdk 'localJdk'
    }

    stages {
        stage('Git checkout') {
            steps {
                echo 'cloning the application code...'
                git branch: 'main', url: 'https://github.com/ndengueche/jenkins-CI-CD-pipeline-project.git'
                sh 'mvn --version'

            }
        }

        stage('Build') {
            steps {
               sh 'java --version'
               sh 'mvn clean package'

            }

            post {
                success {
                    echo 'achiving the package...'
                    archiveArtifacts artifacts: '**/*.war', followSymlinks: false
            }
        }

    }

     stage('Unit Test'){
        steps {
            sh 'mvn test'
        }
    }
    stage('Integration Test'){
        steps {
          sh 'mvn verify -DskipUnitTests'
        }
    }
    stage ('Checkstyle Code Analysis'){
        steps {
            sh 'mvn checkstyle:checkstyle'
        }
        post {
            success {
                echo 'Generated Analysis Result'
            }
        }
    }

    stage ('SonarQube scanning'){
        steps {

            withSonarQubeEnv('SonarQube') {

            sh """
            mvn sonar:sonar \
              -Dsonar.projectKey=maven-project \
              -Dsonar.host.url=http://172.31.15.49:9000 \
              -Dsonar.login=5ca8c85a27dac854f4b00eb69888375e809af1c3

            """
            }
        }

    }

    stage ('upload artifacts to nexus') {
        steps {
            sh 'mvn clean deploy -DskipTests'
        }
    }

    stage('Deploy to DEV') {
          environment {
            HOSTS = "dev"
      }
        steps {
        sh 'ls'
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
     }

     stage ('Deploy to STAGE env') {
        environment {
            HOSTS = "stage"
        }
          steps {
        sh 'ls'
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
    }

    stage ('Approval') {
        steps {
            input ('Do u want to proceed?')
        }
    }

     stage ('Deploy to PROD env') {
        environment {
            HOSTS = "prod"
        }
         steps {
        sh 'ls'
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
    }

    }

     post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#my-notification-channel',
            color: COLOR_MAP[currentBuild.currentResult],
            message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
    }
  }
}
