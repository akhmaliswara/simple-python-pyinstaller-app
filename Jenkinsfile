node {
    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }

    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
        }
    }

    try {
        junit 'test-reports/results.xml'
    } catch (err) {
        // Handling possible errors or exceptions related to JUnit
        echo "Failed to process JUnit results: ${err}"
    }

    stage('Manual Approval') {
        // Add Manual Approval
        input "Proceed with Deliver stage?"
    }
    
    stage('Deploy') {
        env.VOLUME = "${pwd()}/sources:/src"
        env.IMAGE = 'cdrx/pyinstaller-linux:python2'

        dir(env.BUILD_ID) {
            unstash('compiled-results')
            sh "docker run --rm -v ${env.VOLUME} ${env.IMAGE} 'pyinstaller -F add2vals.py'"
        }

        // Delay for 1 minute
        sleep(time: 1, unit: 'MINUTES')

        if (currentBuild.result == 'SUCCESS') {
            archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
            sh "docker run --rm -v ${env.VOLUME} ${env.IMAGE} 'rm -rf build dist'"
        }
    }
}

