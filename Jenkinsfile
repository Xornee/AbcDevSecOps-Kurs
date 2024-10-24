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
                    git credentialsId: 'github-pat', url: 'https://github.com/Xornee/AbcDevSecOps-Kurs/', branch: 'main'
                }
            }
        }
        stage('Prepare') {
            steps {
                sh 'mkdir -p results/'
            }
        }
        stage('[ZAP] Baseline passive-scan') {
            steps {
                sh 'mkdir -p results/'
                sh '''
                    docker run --name juice-shop -d --rm \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                    sleep 15
                '''
                sh '''
                    docker run --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v /home/smytych/DevSecOps/abcd-lab/resources/DAST/zap:/zap/wrk/:rw \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                        "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive_scan.yaml" \
                        || true
                '''
            }
            post {
                always {
                    sh '''
                        docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                        docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                        docker stop zap juice-shop
                        docker rm zap
                    '''
                }
            }
        }
        stage('[OSV-Scanner] Dependency Scan') {
            steps {
                sh 'mkdir -p results/'
                sh '''
                    docker run --rm -v ${WORKSPACE}:/data \
                        ghcr.io/google/osv-scanner:latest \
                        --lockfile /data/package-lock.json \
                        --json > results/osv_scan_report.json
                '''
            }
        }
    }
    post {
        always {
            defectDojoPublisher(
                artifact: 'results/zap_xml_report.xml', 
                productName: 'Juice Shop', 
                scanType: 'ZAP Scan', 
                engagementName: 'szymon.mytych@protonmail.com'
            )
            defectDojoPublisher(
                artifact: 'results/osv_scan_report.json', 
                productName: 'Juice Shop', 
                scanType: 'OSV-Scanner JSON', 
                engagementName: 'szymon.mytych@protonmail.com'
            )
        }
    }
}
