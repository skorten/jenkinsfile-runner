/* Only keep the 10 most recent builds. */
properties([[$class: 'BuildDiscarderProperty',
                strategy: [$class: 'LogRotator', numToKeepStr: '10']]])

// TODO: Move it to Jenkins Pipeline Library

def branchName = currentBuild.projectName
def buildNumber = currentBuild.number

/* These platforms correspond to labels in ci.jenkins.io, see:
 *  https://github.com/jenkins-infra/documentation/blob/master/ci.adoc
 */

Map branches = [:]

for (int i = 0; i < platforms.size(); ++i) {
    String label = platforms[i]
    branches[label] = {
        node(label) {
            timestamps {
                ws("platform_${label}_${branchName}_${buildNumber}") {
                    stage('Checkout') {
                        checkout scm
                    }

                    stage('Build') {
                        timeout(60) {
                            infra.runMaven(['clean', 'verify', '-Dmaven.test.failure.ignore=true', '-Denvironment=test', '-Ppackage-app,package-vanilla,jacoco'])
                        }
                    }
                  
                    stage('SLAnalyze') {
                        dir(".") {
                            sh '/usr/local/bin/sl analyze --app HelloShiftLeft --java target/hello-shiftleft-0.0.1.jar'
                        }
                    }
                }
            }
        }
    }
}

/* Execute our platforms in parallel */
parallel(branches)

