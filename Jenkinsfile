node {
    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }
    stage('Test') {
        try {
            docker.image('qnib/pytest').inside {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
        } finally {
           junit 'test-reports/results.xml'
        }
    }
    stage('Deploy') {
        try {
            dir(path: env.BUILD_ID) { 
                unstash(name: 'compiled-results') 
                sh 'docker run --rm -v "$(pwd)/sources:/src" cdrx/pyinstaller-linux:python2 "pyinstaller -F add2vals.py"' 
            }

            archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals" 
            sh 'docker run --rm -v "$(pwd)/sources:/src" cdrx/pyinstaller-linux:python2 "rm -rf build dist"'
            sh 'sleep 60'
        } finally {
            echo 'Done'
        }
    }
}