'''
In Jenkins, SCM stands for "Source Code Management". 
This option instructs Jenkins to obtain your Pipeline 
from Source Control Management (SCM), which will be your locally 
cloned Git repository. 

We want Jenkins pull the SCM:
Jenkins will reach out to GitHub and if there are any changes 
on the repository it is going to go through the build. 

We want to pull every 2 min: once every 2 min if there is new
code on GitHub, it is going to run the build. 
'''
pipeline {
    agent any

    triggers {
        pollSCM '*/2 * * * *' // every 2 min
    }

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
        stage('Deploy') {
            steps {
                echo 'Deploying...'
                }
        }
    }
}
