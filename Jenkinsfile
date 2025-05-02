pipeline {
    agent any

    environment {
        VENV_DIR = 'venv-checkov'
        CUSTOM_POLICIES = 'custom_policies'
        RENDERED_DIR = 'rendered'
        CHECKOV_OUTPUT = 'checkov_output.txt'
    }

    stages {
        stage('Setup Python Virtual Env') {
            steps {
                sh '''
                python3 -m venv ${VENV_DIR}
                . ${VENV_DIR}/bin/activate
                pip install --upgrade pip
                pip install checkov
                '''
            }
        }

        stage('Render Helm Charts') {
            steps {
                sh '''
                mkdir -p ${RENDERED_DIR}
                helm template ghost-blog ./ghost-blog -n default > ${RENDERED_DIR}/all.yaml
                '''
            }
        }

        stage('Run Checkov Custom Policies Only') {
            steps {
                sh '''
                . ${VENV_DIR}/bin/activate
                echo "👉 Running Checkov with custom policies only..." | tee ${CHECKOV_OUTPUT}

                # Only run Checkov with valid MASA_* policies
                checkov \
                  -d ${RENDERED_DIR} \
                  --framework kubernetes \
                  --external-checks-dir ${CUSTOM_POLICIES} \
                  --check MASA_K8S_001,MASA_K8S_002,MASA_K8S_003,MASA_K8S_004,MASA_K8S_005,MASA_K8S_006,MASA_K8S_007,MASA_K8S_008,MASA_K8S_009,MASA_K8S_010,MASA_K8S_011,MASA_K8S_012,MASA_K8S_013,MASA_K8S_014,MASA_K8S_015 \
                  | tee -a ${CHECKOV_OUTPUT}
                '''
            }
        }

        stage('Checkov Result Summary') {
            steps {
                sh 'cat ${CHECKOV_OUTPUT}'
            }
        }
    }

    post {
        failure {
            echo "❌ Pipeline failed!"
        }
        success {
            echo "✅ Checkov custom policy scan passed."
        }
    }
}
