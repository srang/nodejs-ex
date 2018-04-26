// path of the template to use
def templatePath = 'https://raw.githubusercontent.com/openshift/nodejs-ex/master/openshift/templates/nodejs-mongodb.json'
// name of the template that will be created
def templateName = 'nodejs-mongodb-example'
// NOTE, the "pipeline" directive/closure from the declarative pipeline syntax needs to include, or be nested outside,
// and "openshift" directive/closure from the OpenShift Client Plugin for Jenkins.  Otherwise, the declarative pipeline engine
// will not be fully engaged.
pipeline {
    agent {
      node {
        // spin up a node.js slave pod to run this build on
        label 'nodejs'
      }
    }
    options {
        // set a timeout of 20 minutes for this pipeline
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
        stage('build') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject() {
                            def buildConfig = openshift.selector("bc", "misc-data")
                            buildConfig.startBuild()
                        }
                    }
                } // script
            } // steps
        } // stage
        stage('deploy') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject() {
                            def rm = openshift.selector("dc", "misc-data").rollout()
                            openshift.selector("dc", "misc-data").related('pods').untilEach(1) {
                                return (it.object().status.phase == "Running")
                            }
                        }
                    }
                } // script
            } // steps
        } // stage
        stage('tag') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject() {

                            openshift.tag("misc-data:latest", "misc-data:stage")
                        }
                    }
                } // script
            } // steps
        } // stage
    } // stages
} // pipeline