//!/usr/bin/env groovy

// Only one build running at a time, stop prior build if new build starts
def buildNumber = BUILD_NUMBER as int; if (buildNumber > 1) milestone(buildNumber - 1); milestone(buildNumber) // Thanks to jglick

properties([
    buildDiscarder(logRotator(numToKeepStr: '15')),
    disableResume(),
    durabilityHint('PERFORMANCE_OPTIMIZED'),
    pipelineTriggers([cron('H H/8 * * *')]), // Run once every 8 hours (three times a day)
])

// Define the sequential stages and the parallel steps inside each stage
def sequentialStages = [:]
sequentialStages['Tool'] = [ 'java', 'maven', 'maven-windows', 'maven-11', 'maven-11-windows', 'ruby', ]
sequentialStages['OS & Java'] = [ 'linux', 'windows', 'jdk8', 'jdk11', ]
sequentialStages['Processor'] = [ 'arm64', 'amd64', 'ppc64le', 's390x', ]
sequentialStages['Docker'] = [ 'arm64docker', 'docker', 'ppc64ledocker', 's390xdocker', 'docker', 'docker-windows', ]
sequentialStages['Memory'] = [ 'highmem', 'highram', ]
sequentialStages['Cloud & Orchestrator'] = [ 'aws', 'ppc64le', 's390x', 'kubernetes', 'vm', ] // 'aci', 'azure' not included currently

// Generate a parallel step for each label in labels
def generateParallelSteps(labels) {
    def parallelNodes = [:]
    for (unboundLabel in labels) {
        def label = unboundLabel // Bind label before the closure
        parallelNodes[label] = {
            node(label) {
                if (isUnix()) {
                    sh 'uname -a'
                } else {
                    bat 'set | findstr Windows'
                }
            }
        }
    }
    return parallelNodes
}

timeout(unit: 'MINUTES', time:29) {
    for (unboundStage in sequentialStages) {
        def boundStage = unboundStage
        stage(boundStage.key) {
            parallel generateParallelSteps(sequentialStages[boundStage.key])
        }
    }
}
