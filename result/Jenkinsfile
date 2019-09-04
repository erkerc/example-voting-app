pipeline {
  agent {
    node {
      label 'nodejs' 
    }
  }
  environment {
    appName = 'result'
  }

  options {
    timeout(time: 20, unit: 'MINUTES') 
  }
  stages {
    stage('preamble') {
        steps {
            script {
                openshift.withCluster() {
                    openshift.withProject() {
                        echo "Using project: ${openshift.project()}"
                    }
                }
            }
        }
    }
    stage('cleanup') {
      steps {
        script {
           /* openshift.withCluster() {
                openshift.withProject() {
                  openshift.selector("all", [ template : appName ]).delete() 
                  if (openshift.selector("secrets", appName).exists()) { 
                    openshift.selector("secrets", appName).delete()
                  }
                }
            }
        }*/

          echo "Step: [cleanup] project: ${openshift.project()} app:${appName}"

      }
    }
    stage('create') {
      steps {

      script {
           /* openshift.withCluster() {
                openshift.withProject() {
                  openshift.newApp(templatePath) 
                }
            }*/
            echo "Step: [create] project: ${openshift.project()} app:${appName}"

        }
      }
    }
    stage('build') {
      steps {
        script {
            echo "Building project: ${openshift.project()} app:${appName}"

            openshift.withCluster() {
                openshift.withProject() {
                  def builds = openshift.selector("bc", appName).related('builds')
                  timeout(5) { 
                    builds.untilEach(1) {
                      return (it.object().status.phase == "Complete")
                    }
                  }
                }

              echo "Built project: ${openshift.project()} app:${appName}"

            }
        }
      }
    }
    stage('deploy') {
      steps {
        script {
            echo "Deploying project: ${openshift.project()} app:${appName}"

            openshift.withCluster() {
                openshift.withProject() {
                  def rm = openshift.selector("dc", appName).rollout().latest()
                  timeout(5) { 
                    openshift.selector("dc", appName).related('pods').untilEach(1) {
                      return (it.object().status.phase == "Running")
                    }
                  }
                }
            }

            echo "Deployed project: ${openshift.project()} app:${appName}"

        }
      }
    }
    stage('tag') {
      steps {
        script {
            echo "Tagging project: ${openshift.project()} app:${appName}"

            openshift.withCluster() {
                openshift.withProject() {
                  openshift.tag("${appName}:latest", "${appName}-staging:latest") 
                }
            }
            echo "Tagged project: ${openshift.project()} app:${appName}"

        }
      }
    }
  }
}
