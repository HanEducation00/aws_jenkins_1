### 1. Jenkins Pipeline Script kısmına yazacağımız kod. Bu kodu "jenkinsfile" adında dosyada tutabiliriz.

```
pipeline {
    agent any

    triggers {
        githubPush()  // GitHub'dan gelen push'ları dinler
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/KuserOguzHan/githup_jenkins_1.git'
            }
        }

        stage('Build and Run') {
            steps {
                sh 'chmod +x jenkins-script.sh'
                sh './jenkins-script.sh'
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}

```
- jenkins-script.sh dosyası
```
sleep 5
echo "####################################"
sleep 5
echo "Jenkis bağlantsı başarlı !!!"
sleep 5
echo "####################################"
sleep 3
```
### 2.FastApi 1.Adım

```
pipeline {
    agent any

    triggers {
        githubPush()  // GitHub'dan gelen push'ları dinler
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/KuserOguzHan/githup_jenkins_1.git'
            }
        }

        stage('Virtualenv and Requirements') {
            steps {
                script {
                    echo 'Setting up Python virtual environment and installing requirements...'
                    sh '''#!/bin/bash
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install -r requirements.txt
                    '''
                }
                echo 'Python environment is set up and requirements are installed'
            }
        }

        stage('Run FastAPI Application') {
            steps {
                script {
                    echo 'Starting FastAPI application with Uvicorn...'
                    sh '''#!/bin/bash
                    . venv/bin/activate
                    uvicorn main:app --host 0.0.0.0 --port 8002
                    '''
                }
                echo 'Uvicorn is running'
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
```





