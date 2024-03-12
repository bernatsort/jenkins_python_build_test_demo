pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: 'main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/bernatsort/jenkins_python_build_test_demo.git']])
            }
        }
        
        stage('Build') {
            steps {
                git branch: 'main', url: 'https://github.com/bernatsort/jenkins_python_build_test_demo.git'
                // Change to the 'source' directory
                dir('/var/jenkins_home/workspace/Python Build and Test Demo/source') {
                    // Run your app.py using the appropriate command
                    sh 'python3 inc_dec.py'
                }
            }
        }
        stage('Test') {
            steps {
                // Change to the root directory of the project
                dir('/var/jenkins_home/workspace/Python Build and Test Demo') {
                    // Run pytest to execute tests
                    sh 'python3 -m pytest tests/test_pytest.py'
                }
            }
        }
    }
}