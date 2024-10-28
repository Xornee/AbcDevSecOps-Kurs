pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'github-pat', url: 'https://github.com/Xornee/AbcDevSecOps-Kurs/', branch: 'main', extensions: [
                        [$class: 'CloneOption', noTags: false, shallow: false, depth: 0]
                    ]
                }
            }
        }
        stage('Prepare') {
            steps {
                sh 'mkdir -p results/'
            }
        }
        stage('Debug') {
            steps {
                sh 'ls -la'
                sh 'ls -la .git'
            }
        }

        // stage('[ZAP] Baseline passive-scan') {
        //     steps {
        //         sh 'mkdir -p results/'
        //         sh '''
        //             docker run --name juice-shop -d --rm \
        //                 -p 3000:3000 \
        //                 bkimminich/juice-shop
        //             sleep 15
        //         '''
        //         sh '''
        //             docker run --name zap \
        //                 --add-host=host.docker.internal:host-gateway \
        //                 -v /home/smytych/DevSecOps/abcd-lab/resources/DAST/zap:/zap/wrk/:rw \
        //                 -t ghcr.io/zaproxy/zaproxy:stable bash -c \
        //                 "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive_scan.yaml" \
        //                 || true
        //         '''
        //     }
        //     post {
        //         always {
        //             sh '''
        //                 docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
        //                 docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
        //                 docker stop zap juice-shop
        //                 docker rm zap
        //             '''
        //         }
        //     }

        // }
        // stage('[OSV-Scanner] Dependency Scan') {
        //     steps {
        //         sh 'mkdir -p results/'
        //         sh '''
        //             docker run --rm -v /home/smytych/DevSecOps/AbcDevSecOps-Kurs:/data \
        //                 ghcr.io/google/osv-scanner:latest \
        //                 --lockfile /data/package-lock.json \
        //                 --json > results/osv_scan_report.json \
        //                 || true
        //         '''
        //     }
        // }
        stage('[TruffleHog] Secret Scan') {
            steps {
                sh 'mkdir -p results/'
                sh '''
                    docker run --rm -v $PWD:/data \
                        trufflesecurity/trufflehog:latest \
                        git file:///data --branch main --json > results/trufflehog_report.json || true
                '''
            }
        }
    }
    post {
        always {
            // defectDojoPublisher(
            //     artifact: 'results/zap_xml_report.xml', 
            //     productName: 'Juice Shop', 
            //     scanType: 'ZAP Scan', 
            //     engagementName: 'szymon.mytych@protonmail.com'
            // )
            // defectDojoPublisher(
            //     artifact: 'results/osv_scan_report.json', 
            //     productName: 'Juice Shop', 
            //     scanType: 'OSV Scan', 
            //     engagementName: 'szymon.mytych@protonmail.com'
            // )
            defectDojoPublisher(
                artifact: 'results/trufflehog_report.json', 
                productName: 'Juice Shop', 
                scanType: 'Trufflehog Scan', 
                engagementName: 'szymon.mytych@protonmail.com'
            )
        }
    }
}
