pipeline {
    agent any
    
    environment {
        DOJO_URL = "http://host.docker.internal:8081"
        DOJO_API_KEY = ""
        PRODUCT_NAME = "" 
    }

    stages {
        stage('0. Ortam Hazırlığı (Docker CLI)') {
            steps {
                echo 'Docker CLI aracı indiriliyor...'
                sh '''
                    curl --retry 3 --retry-delay 5 -fsSL https://download.docker.com/linux/static/stable/x86_64/docker-27.5.1.tgz -o docker.tgz
                    tar xzvf docker.tgz
                '''
            }
        }

        stage('1. Secret Scanning (Gitleaks)') {
            steps {
                echo 'Kod içinde sızıntı kontrolü yapılıyor...'
                sh '''
                    curl -sL https://github.com/gitleaks/gitleaks/releases/download/v8.18.1/gitleaks_8.18.1_linux_x64.tar.gz -o gitleaks.tar.gz
                    tar -xzf gitleaks.tar.gz
                    ./gitleaks detect --no-git --source="." -v --report-format json --report-path gitleaks-results.json || true
                '''
            }
        }

        stage('2. SAST (SonarQube)') {
            steps {
                echo 'Statik Kod Analizi (SAST) başlatılıyor...'
                script {
                    withSonarQubeEnv('sonar-server') {
                        def scannerHome = tool 'sonar-scanner'
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=juice-shop -Dsonar.sources=."
                    }
                }
            }
        }

        stage('3. SCA & Container Security (Trivy)') {
            steps {
                echo 'Juice Shop Docker İmaj Zafiyetleri aranıyor...'
                sh '''
                    ./docker/docker run --rm ghcr.io/aquasecurity/trivy:latest image --format json bkimminich/juice-shop > trivy-results.json || true
                '''
            }
        }

        stage('4. Deployment (Test Ortamı)') {
            steps {
                echo 'Juice Shop ayağa kaldırılıyor...'
                sh '''
                    ./docker/docker rm -f juice-shop-test || true
                    ./docker/docker run -d --name juice-shop-test --network devsecops-net -p 3000:3000 bkimminich/juice-shop
                '''
                sleep 20 
            }
        }

        stage('5. DAST (OWASP ZAP & Nuclei)') {
            parallel {
                stage('ZAP Davranışsal Tarama') {
                    steps {
                        echo 'ZAP Baseline Taraması Başlıyor...'
                        sh '''
                            ./docker/docker rm -f zap-scanner || true
                            ./docker/docker volume rm zap_wrk || true
                            
                            ./docker/docker volume create zap_wrk
                            
                            # ZAP XML formatında rapor üretiyor
                            ./docker/docker run --name zap-scanner -u root --network devsecops-net -v zap_wrk:/zap/wrk:rw ghcr.io/zaproxy/zaproxy:nightly zap-baseline.py -t http://juice-shop-test:3000 -x zap-report.xml || true
                            
                            # XML raporunu çekiyoruz
                            ./docker/docker cp zap-scanner:/zap/wrk/zap-report.xml ./zap-report.xml || true
                            
                            ./docker/docker rm -f zap-scanner || true
                            ./docker/docker volume rm zap_wrk || true
                        '''
                    }
                }
                stage('Nuclei Zafiyet Taraması') {
                    steps {
                        echo 'Nuclei şablon tabanlı saldırı simülasyonu...'
                        sh '''
                            ./docker/docker run --rm --network devsecops-net projectdiscovery/nuclei:latest -u http://juice-shop-test:3000 -jsonl -silent -rl 50 -c 10 > nuclei-results.json || true
                        '''
                    }
                }
            }
        }

        stage('6. Zafiyet Yönetimi (DefectDojo Raporlaması)') {
            steps {
                echo "Tüm raporlar Dojo'ya postalanıyor..."
                
                sh '''
                    if [ -f "gitleaks-results.json" ] && [ -s "gitleaks-results.json" ]; then
                        curl -X POST "${DOJO_URL}/api/v2/import-scan/" \
                        -H "Authorization: Token ${DOJO_API_KEY}" \
                        -F "scan_type=Gitleaks Scan" \
                        -F "file=@gitleaks-results.json" \
                        -F "product_name=${PRODUCT_NAME}" \
                        -F "engagement_name=Secret Analysis" \
                        -F "auto_create_context=true"
                    fi
                '''

                sh '''
                    if [ -f "trivy-results.json" ] && [ -s "trivy-results.json" ]; then
                        curl -X POST "${DOJO_URL}/api/v2/import-scan/" \
                        -H "Authorization: Token ${DOJO_API_KEY}" \
                        -F "scan_type=Trivy Scan" \
                        -F "file=@trivy-results.json" \
                        -F "product_name=${PRODUCT_NAME}" \
                        -F "engagement_name=Trivy SCA Analysis" \
                        -F "auto_create_context=true"
                    fi
                '''

                sh '''
                    if [ -f "zap-report.xml" ] && [ -s "zap-report.xml" ]; then
                        curl -X POST "${DOJO_URL}/api/v2/import-scan/" \
                        -H "Authorization: Token ${DOJO_API_KEY}" \
                        -F "scan_type=ZAP Scan" \
                        -F "file=@zap-report.xml" \
                        -F "product_name=${PRODUCT_NAME}" \
                        -F "engagement_name=ZAP DAST Analysis" \
                        -F "auto_create_context=true"
                    fi
                '''

                sh '''
                    if [ -f "nuclei-results.json" ] && [ -s "nuclei-results.json" ]; then
                        curl -X POST "${DOJO_URL}/api/v2/import-scan/" \
                        -H "Authorization: Token ${DOJO_API_KEY}" \
                        -F "scan_type=Nuclei Scan" \
                        -F "file=@nuclei-results.json" \
                        -F "product_name=${PRODUCT_NAME}" \
                        -F "engagement_name=Nuclei DAST Analysis" \
                        -F "auto_create_context=true"
                    fi
                '''
                
                echo 'Bütün araçlar çalıştı, raporlar OWASP Juice Shop projesine iletildi.'
            }
        }
    }
}