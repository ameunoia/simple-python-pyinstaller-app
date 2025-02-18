// pipeline {
//     agent any
//     stages {
//         stage('Build') {
//             agent {
//                 docker {
//                     image 'python:2-alpine'
//                 }
//             }
//             steps {
//                 sh 'python -m py_compile sources/add2vals.py sources/calc.py'
//             }
//         }
//         stage('Test') {
//             agent {
//                 docker {
//                     image 'qnib/pytest'
//                 }
//             }
//             steps {
//                 sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
//             }
//             post {
//                 always {
//                     junit 'test-reports/results.xml'
//                 }
//             }
//         }
//         stage('Deliver') {
//             agent {
//                 docker {
//                     image 'cdrx/pyinstaller-linux:python2'
//                 }
//             }
//             steps {
//                 sh 'pyinstaller --onefile sources/add2vals.py'
//             }
//             post {
//                 success {
//                     archiveArtifacts 'dist/add2vals'
//                 }
//             }
//         }
//     }
// }

node {
    stage('Checkout') {
        checkout scm
    }
    stage('Build') {
        docker.image('python:3.9-alpine').inside('-p 3000:3000') {
            echo 'Building Application'
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        }
    }
    stage('Test') {
        docker.image('qnib/pytest').inside {
            echo 'running test'
            sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
        }
        step([$class: 'JUnitResultArchiver', testResults: 'test-reports/results.xml'])
    }
    stage('Manual Approval') {
            input message: 'Lanjutkan ke tahap Deploy? (Klik "Proceed" untuk melanjutkan)'
        }
    stage('Deploy') {
        try {
            docker.image('node:18-alpine').inside {
                echo 'Deploting to Vercel'
                // sh 'pyinstaller --onefile sources/add2vals.py'
                sh '''
                    npx vercel --prod --token PwAHxxWNmkJ0wxOhoOKbvnwr --yes --force
                    sleep(time:1, unit:"MINUTES")
                    npx vercel remove submission-cicd-pipeline-argasatya16-python --token PwAHxxWNmkJ0wxOhoOKbvnwr --yes
                '''
            }
        } catch (exc) {
            echo "failed deploy stage"
        }
    }
}