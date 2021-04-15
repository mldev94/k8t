library("base")

pipeline {
  agent {
    kubernetes {
      defaultContainer 'python'
      yamlFile '.jenkins/python.yaml'
    }
  }

  options {
    timeout(time: 2, unit: 'HOURS')
    buildDiscarder(
      logRotator(
        daysToKeepStr: '14',
        numToKeepStr: '100',
        artifactDaysToKeepStr: '30',
      ))
  }

  stages {
    stage('setup') {
      steps {
        sh 'apk add git gcc musl-dev python3-dev linux-headers libffi-dev libressl-dev build-base cargo'
        sh 'curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh'
        sh 'pip install --upgrade pip tox'
      }
    }

    stage('test') {
      steps {
        withStatus(context: 'ci/test') {
          sh 'tox'
        }
      }
    }

    stage('build') {
      when {
        anyOf {
          branch 'master'
          buildingTag()
        }
      }

      steps {
        withStatus(context: 'ci/build') {
          sh 'pip install --upgrade setuptools wheel'
          sh 'python3 setup.py sdist bdist_wheel'
        }
      }
    }

    stage('release') {
      when {
        buildingTag()
      }

      steps {
        withStatus(context: 'ci/release') {
          sh 'pip install --upgrade twine'
          withCredentials([usernamePassword(credentialsId: 'pypi', usernameVariable: 'TWINE_USERNAME', passwordVariable: 'TWINE_PASSWORD')]) {
            sh 'twine upload dist/*'
          }
        }
      }
    }
  }
}
