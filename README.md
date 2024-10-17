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
                git branch: 'main', url: 'https://github.com/HanEducation00/aws_jenkins_1'
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

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/HanEducation00/aws_jenkins_1'
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install -r requirements.txt
                    '''
                }
            }
        }

        stage('Run FastAPI Application') {
            steps {
                script {
                    sh '''
                    . venv/bin/activate
                    uvicorn main:app --host 0.0.0.0 --port 8002
                    '''
                }
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





