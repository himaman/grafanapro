pipeline {
    agent {
        label "any node"
    }
    stages {
        stage("install prometheus") {
            steps {
                script {
                    def jenkinsWorkspace = pwd()

                    echo "Jenkins Workspace Path: ${jenkinsWorkspace}"

                    // Download and extract the latest Prometheus release
                    sh '''
                        latest_version=$(curl -s https://api.github.com/repos/prometheus/prometheus/releases/latest | jq -r .tag_name)
                        curl -LO https://github.com/prometheus/prometheus/releases/download/${latest_version}/prometheus-${latest_version}.linux-amd64.tar.gz
                        tar -xvf prometheus-${latest_version}.linux-amd64.tar.gz
                        mv prometheus-${latest_version}.linux-amd64 ${jenkinsWorkspace}/prometheus-files
                    '''
                }
            }
        }
        stage("create systemd service") {
            steps {
                script {
                    def jenkinsWorkspace = pwd()

                    sh '''
                        cat <<EOL | sudo tee /etc/systemd/system/prometheus.service
                        [Unit]
                        Description=Prometheus
                        After=network.target

                        [Service]
                        ExecStart=${jenkinsWorkspace}/prometheus-files/prometheus --config.file=${jenkinsWorkspace}/prometheus-files/prometheus.yml
                        Restart=always

                        [Install]
                        WantedBy=default.target
                        EOL
                    '''
                    sh 'sudo systemctl daemon-reload'
                    sh 'sudo systemctl enable prometheus'
                    // Start and enable the Prometheus service
                    sh 'sudo systemctl start prometheus'

                }
            }
        }
        stage("verify prometheus status") {
            steps {
                script {
                    // Check Prometheus status
                    sh 'sudo systemctl status prometheus'
                }
            }
        }
    }
}
