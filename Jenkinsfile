pipeline {
    agent any
    environment {
        VENV_PATH = 'venv-checkov'
    }

    stages {
        stage('Set Up Virtual Environment and Install Checkov') {
            steps {
                sh '''
                    apt update && apt install python3.11-venv -y
                    python3 -m venv ${VENV_PATH}
                    . ${VENV_PATH}/bin/activate
                    ${VENV_PATH}/bin/pip install --upgrade pip
                    ${VENV_PATH}/bin/pip install checkov
                    curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
                '''
            }
        }

        stage('Render Helm Charts') {
            steps {
                sh '''
                    echo "🧹 Cleaning previous rendered files..."
                    rm -rf rendered/
                    mkdir -p rendered

                    echo "🔧 Rendering Helm charts into split files..."
                    # Check if the ghost-blog directory exists before proceeding
                    if [ -d "./ghost-blog" ]; then
                        find ./ghost-blog -name "Chart.yaml" | while read chart_file; do
                            chart_dir=$(dirname "$chart_file")
                            chart_name=$(basename "$chart_dir" | tr '[:upper:]' '[:lower:]' | tr -cd 'a-z0-9-')
                            mkdir -p rendered/${chart_name}
                            helm template ${chart_name} ${chart_dir} --output-dir rendered/${chart_name}
                        done
                    else
                        echo "Error: './ghost-blog' directory does not exist."
                        exit 1
                    fi
                '''
            }
        }

        stage('Run Checkov Custom Policies') {
            steps {
                sh '''
                    echo "👉 Running Checkov with custom policies only..."
                    rm -f checkov_output.txt

                    ${VENV_PATH}/bin/checkov \
                        -d rendered \
                        --framework kubernetes \
                        --external-checks-dir custom_policies \
                        --run-all-external-checks \
                        | tee -a checkov_output.txt
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'checkov_output.txt', onlyIfSuccessful: false
        }
    }
}
