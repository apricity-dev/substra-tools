pipeline {
  options {
    timestamps ()
    timeout(time: 1, unit: 'HOURS')
    buildDiscarder(logRotator(numToKeepStr: '5'))
    skipDefaultCheckout true
  }

  agent none

  stages {
    stage('Abort previous builds'){
      steps {
        milestone(Integer.parseInt(env.BUILD_ID)-1)
        milestone(Integer.parseInt(env.BUILD_ID))
      }
    }

    stage('Test & Build') {
      parallel {

        stage('Test') {
          agent {
            kubernetes {
              label 'python'
              defaultContainer 'python'
              yaml """
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: python
                    image: python:3.7
                    command: [cat]
                    tty: true
                """
            }
          }

          steps {
            dir("substratools") {
              checkout scm
              sh "pip install -e .[test]"
              sh "python setup.py test"
            }
          }
        }

        stage('Build') {
          agent {
            kubernetes {
              label 'substratools-kaniko'
              yamlFile '.cicd/agent-kaniko.yaml'
            }
          }

          steps {
            container(name:'kaniko', shell:'/busybox/sh') {
              checkout scm
              sh '''#!/busybox/sh
                /kaniko/executor -f `pwd`/Dockerfile -c `pwd` -d "eu.gcr.io/substra-208412/substratools:$GIT_COMMIT"
              '''
            }
          }
        }
      }
    }
  }
}