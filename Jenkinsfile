library identifier: "pipeline-library@master",
retriever: modernSCM(
  [
    $class: "GitSCMSource",
    remote: "https://github.com/redhat-cop/pipeline-library.git"
  ]
)

openshift.withCluster() {
  env.POM_FILE = env.BUILD_CONTEXT_DIR ? "${env.BUILD_CONTEXT_DIR}/pom.xml" : "pom.xml"
  env.TARGET = env.BUILD_CONTEXT_DIR ? "${env.BUILD_CONTEXT_DIR}/target" : "target"
  env.APP_NAME = "${JOB_NAME}".replaceAll(/-build.*/, '')
  env.BUILD = openshift.project()
  env.DEV = "${APP_NAME}-dev"
  env.STAGE = "${APP_NAME}-stage"
  env.PROD = "${APP_NAME}-prod"
  echo "Starting Pipeline for ${APP_NAME}..."
  
}

pipeline {
  agent {
    label 'maven'
  }

  stages {
    stage('Git Checkout') {
      steps {
        git url: "${APPLICATION_SOURCE_REPO}", branch: "${APPLICATION_SOURCE_REF}"
      }
    }

    stage('Build') {
      steps {
        sh "mvn -B clean install -DskipTests=true -f ${POM_FILE}"
      }
    }

    stage('Unit Test') {
      steps {
        sh "mvn -B test -f ${POM_FILE}"
      }
    }

    stage('Build Container Image') {
      steps {
        sh 'echo $PWD'
        binaryBuild(projectName: env.BUILD, buildConfigName: env.APP_NAME, "./")
      }
    }

    stage('Promote from Build to Dev') {
      steps {
        tagImage(sourceImageName: env.APP_NAME, sourceImagePath: env.BUILD, toImagePath: env.DEV)
      }
    }

   
    stage('Promotion gate (Stage)') {
      steps {
        script {
          input message: 'Promote Application to Stage?'
        }
      }
    }

    stage('Promote from Dev to Stage') {
      steps {
        tagImage(sourceImageName: env.APP_NAME, sourceImagePath: env.DEV, toImagePath: env.STAGE)
      }
    }
  }
}

println "Application ${env.APP_NAME} is now in Stage!"
