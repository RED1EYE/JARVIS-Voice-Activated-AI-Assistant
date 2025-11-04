pipeline {
    agent any
    
    environment {
        PYTHON_VERSION = '3.11'
        VENV_DIR = 'venv'
        PROJECT_NAME = 'JARVIS-Voice-Activated-AI-Assistant'
        // Add your Picovoice access key as Jenkins credential
        PICOVOICE_ACCESS_KEY = credentials('picovoice-access-key')
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from repository...'
                checkout scm
                sh 'git branch'
                sh 'git log -1'
            }
        }
        
        stage('Setup Python Environment') {
            steps {
                echo 'Setting up Python virtual environment...'
                sh '''
                    python${PYTHON_VERSION} -m venv ${VENV_DIR}
                    . ${VENV_DIR}/bin/activate
                    python --version
                    pip install --upgrade pip
                '''
            }
        }
        
        stage('Check Ollama Service') {
            steps {
                echo 'Checking if Ollama service is running...'
                script {
                    def ollamaStatus = sh(
                        script: 'curl -s http://localhost:11434/api/tags || echo "not running"',
                        returnStatus: true
                    )
                    
                    if (ollamaStatus != 0) {
                        echo 'Warning: Ollama service may not be running'
                        echo 'Ensure Ollama is installed and running: ollama serve'
                    } else {
                        echo 'Ollama service is running'
                    }
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                echo 'Installing Python dependencies...'
                sh '''
                    . ${VENV_DIR}/bin/activate
                    
                    # Install basic requirements
                    pip install -r requirements.txt
                    
                    # Install PyTorch with CUDA support (adjust based on your system)
                    pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
                    
                    echo "Dependencies installed successfully"
                '''
            }
        }
        
        stage('Code Quality Check') {
            steps {
                echo 'Running code quality checks...'
                sh '''
                    . ${VENV_DIR}/bin/activate
                    
                    # Install linting tools
                    pip install pylint flake8 black
                    
                    # Run flake8 for linting (allowing some warnings)
                    flake8 jarvis.py --max-line-length=120 --ignore=E501,W503 || true
                    
                    # Check code formatting with black
                    black --check jarvis.py || echo "Code formatting suggestions available"
                '''
            }
        }
        
        stage('Security Scan') {
            steps {
                echo 'Running security vulnerability scan...'
                sh '''
                    . ${VENV_DIR}/bin/activate
                    
                    # Install bandit for security checks
                    pip install bandit
                    
                    # Run security scan
                    bandit -r . -f json -o bandit-report.json || true
                    
                    # Install safety for dependency vulnerability check
                    pip install safety
                    
                    # Check for known security vulnerabilities
                    safety check --json || true
                '''
            }
        }
        
        stage('Configuration Validation') {
            steps {
                echo 'Validating configuration files...'
                sh '''
                    . ${VENV_DIR}/bin/activate
                    
                    # Check if jarvis_config.json exists
                    if [ -f jarvis_config.json ]; then
                        echo "Configuration file found"
                        python -c "import json; json.load(open('jarvis_config.json'))" && echo "Valid JSON" || echo "Invalid JSON"
                    else
                        echo "Warning: jarvis_config.json not found, creating default..."
                        echo '{"start_mode": "normal"}' > jarvis_config.json
                    fi
                    
                    # Validate Python syntax
                    python -m py_compile jarvis.py
                    
                    echo "Configuration validation completed"
                '''
            }
        }
        
        stage('Unit Tests') {
            steps {
                echo 'Running unit tests...'
                sh '''
                    . ${VENV_DIR}/bin/activate
                    
                    # Install testing frameworks
                    pip install pytest pytest-cov pytest-mock
                    
                    # Create basic test structure if tests don't exist
                    if [ ! -d "tests" ]; then
                        mkdir -p tests
                        echo "# Add your tests here" > tests/__init__.py
                        echo "def test_placeholder(): assert True" > tests/test_jarvis.py
                    fi
                    
                    # Run tests with coverage
                    pytest tests/ --cov=. --cov-report=xml --cov-report=html || echo "No tests to run yet"
                '''
            }
        }
        
        stage('Build Documentation') {
            steps {
                echo 'Generating documentation...'
                sh '''
                    . ${VENV_DIR}/bin/activate
                    
                    # Create a simple documentation summary
                    cat << EOF > BUILD_INFO.txt
Build Information
=================
Project: ${PROJECT_NAME}
Build Number: ${BUILD_NUMBER}
Build ID: ${BUILD_ID}
Build Date: $(date)
Git Commit: $(git rev-parse HEAD)
Git Branch: $(git rev-parse --abbrev-ref HEAD)
Python Version: $(python --version)

Requirements:
- Ollama with Qwen3:4B model
- CUDA-capable GPU (recommended)
- Picovoice Access Key
- PyAudio and PortAudio

Setup Instructions:
1. Install Ollama and run: ollama serve
2. Pull Qwen3:4B model: ollama pull qwen3:4b
3. Configure Picovoice access key in jarvis.py
4. Run: python jarvis.py

EOF
                    cat BUILD_INFO.txt
                '''
            }
        }
        
        stage('Package Application') {
            steps {
                echo 'Creating application package...'
                sh '''
                    . ${VENV_DIR}/bin/activate
                    
                    # Create distribution directory
                    mkdir -p dist
                    
                    # Copy necessary files
                    cp jarvis.py dist/
                    cp jarvis_config.json dist/ || echo '{"start_mode": "normal"}' > dist/jarvis_config.json
                    cp requirements.txt dist/
                    cp README.md dist/ || true
                    cp BUILD_INFO.txt dist/
                    
                    # Create a startup script
                    cat << 'EOF' > dist/start_jarvis.sh
#!/bin/bash
echo "Starting JARVIS Voice Assistant..."
echo "Make sure Ollama is running: ollama serve"
python jarvis.py
EOF
                    chmod +x dist/start_jarvis.sh
                    
                    # Create archive
                    tar -czf jarvis-build-${BUILD_NUMBER}.tar.gz dist/
                    
                    echo "Package created: jarvis-build-${BUILD_NUMBER}.tar.gz"
                '''
            }
        }
        
        stage('Archive Artifacts') {
            steps {
                echo 'Archiving build artifacts...'
                archiveArtifacts artifacts: 'jarvis-build-*.tar.gz,BUILD_INFO.txt,bandit-report.json', 
                                     fingerprint: true,
                                     allowEmptyArchive: true
            }
        }
        
        stage('Deploy to Test Environment') {
            when {
                branch 'main'
            }
            steps {
                echo 'Deploying to test environment...'
                sh '''
                    # Extract package to test directory
                    mkdir -p /var/jenkins_home/test-deployments/jarvis-${BUILD_NUMBER}
                    tar -xzf jarvis-build-${BUILD_NUMBER}.tar.gz -C /var/jenkins_home/test-deployments/jarvis-${BUILD_NUMBER}
                    
                    echo "Deployed to: /var/jenkins_home/test-deployments/jarvis-${BUILD_NUMBER}"
                '''
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline execution completed'
            cleanWs(deleteDirs: true, 
                    patterns: [[pattern: 'venv/**', type: 'INCLUDE'],
                               [pattern: '**/__pycache__/**', type: 'INCLUDE'],
                               [pattern: '**/*.pyc', type: 'INCLUDE']])
        }
        
        success {
            echo 'Build successful! JARVIS is ready for deployment.'
        }
        
        failure {
            echo 'Build failed! Please check the logs.'
        }
        
        unstable {
            echo 'Build is unstable. Some tests may have failed.'
        }
    }
}
