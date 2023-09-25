pipeline {
    agent any 
    environment {
        caminhoPacote = 'target/verademo.war'
        wrapperVersion = '23.8.12.0'
    }
    stages {
        stage('Clean') { 
            steps {
                sh 'rm -rf pipeline-scan-LATEST.zip pipeline-scan.jar'
            }
        }
        stage('Build') { 
            steps {
                sh 'mvn clean package'
                sh 'ls -l target/'
            }
        }
        stage('Veracode SCA - Agent Scan') { 
            steps {
                withCredentials([string(credentialsId: 'SRCCLR_API_TOKEN', variable: 'SRCCLR_API_TOKEN')]) {
                    sh 'curl -sSL https://download.sourceclear.com/ci.sh | bash -s scan --update-advisor --uri-as-name || true'
                }
            }
        }

        stage('Veracode SAST - Sandbox Scan') {
            steps {
                withCredentials([string(credentialsId: 'VID', variable: 'VID'), string(credentialsId: 'VKEY', variable: 'VKEY')]) {
                    veracode applicationName: 'Java-VeraDemo',
                    criticality: 'VeryHigh', 
                    deleteIncompleteScan: 2,
                    createSandbox: true,
                    sandboxName: 'SANDBOX_1',
                    scanName: '${BUILD_TIMESTAMP} - ${BUILD_NUMBER}', 
                    uploadIncludesPattern: '**/**.war', 
                    vid: ${VID}, 
                    vkey: ${VKEY}         
                }
            }
        }

        stage('Veracode SAST - Policy Scan') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'veracode-credentials', passwordVariable: 'VKEY', usernameVariable: 'VID')]) {
                    veracode applicationName: 'Java-VeraDemo',
                    canFailJob: false,
                    criticality: 'VeryHigh', 
                    debug: true, 
                    deleteIncompleteScan: true, 
                    scanName: '${BUILD_TIMESTAMP} - ${BUILD_NUMBER}', 
                    uploadIncludesPattern: '**/**.war', 
                    vid: ${VID}, 
                    vkey: ${VKEY}, 
                    waitForScan: false
                }
            }
        }
        
        stage('Veracode SAST - Pipeline Scan') { 
            steps {
                withCredentials([usernamePassword(credentialsId: 'veracode-credentials', passwordVariable: 'VKEY', usernameVariable: 'VID')]) {
                sh 'curl -sSO https://downloads.veracode.com/securityscan/pipeline-scan-LATEST.zip'
                sh 'unzip -o pipeline-scan-LATEST.zip'
                sh (""" java -jar pipeline-scan.jar \
                    --veracode_api_id "${VID}" \
                    --veracode_api_key "${VKEY}" \
                    --file ${caminhoPacote} \
                    --json_file baseline.json
                    || true """)
                }
            }
        }

        stage('Veracode SAST - Pipeline Scan Baseline File') { 
            steps {
                withCredentials([usernamePassword(credentialsId: 'veracode-credentials', passwordVariable: 'VKEY', usernameVariable: 'VID')]) {
                sh 'curl -sSO https://downloads.veracode.com/securityscan/pipeline-scan-LATEST.zip'
                sh 'unzip -o pipeline-scan-LATEST.zip'
                sh (""" java -jar pipeline-scan.jar \
                    --veracode_api_id "${VID}" \
                    --veracode_api_key "${VKEY}" \
                    --file ${caminhoPacote} \
                    --project_name "Java-VeraDemo"
                    --baseline_file baseline.json
                    """)
                }
            }
        }
    }
}