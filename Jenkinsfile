
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
                        openshift.logLevel(3)
                        openshift.withProject() {
                            def buildConfig = openshift.selector("bc", "misc-data")
                            def build = buildConfig.startBuild()
                            build.describe()
                            build.logs('-f')
                            buildConfig.related('builds').untilEach(1) {
                                return (it.object().status.phase == "Complete")
                            }
                        }
                    }
                } // script
            } // steps
        } // stage
        stage('deploy') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.logLevel(3)
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