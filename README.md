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


### 3. Sonarqube eklentisi.

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

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'SonarQubeScanner'
                SONARQUBE_TOKEN = credentials('jenkins-sonarqube-token')
            }
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh '''
                    . venv/bin/activate
                    ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=aws_jenkins_1 \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://16.16.25.203:9000 \
                        -Dsonar.python.version=3.8 \
                        -Dsonar.login=${SONARQUBE_TOKEN}
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

### 4. SonarQube Quality


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

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'SonarQubeScanner'
                SONARQUBE_TOKEN = credentials('jenkins-sonarqube-token')
            }
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh '''
                    . venv/bin/activate
                    ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=aws_jenkins_1 \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://16.16.25.203:9000 \
                        -Dsonar.python.version=3.8 \
                        -Dsonar.login=${SONARQUBE_TOKEN} \
                        -Dsonar.ce.taskId= \
                        -Dsonar.ce.task.timeout=300s  # Set timeout for SonarQube task
                    '''
                }
            }
        }

        stage('Quality Gate') {
            environment {
                scannerHome = tool 'SonarQubeScanner'
                SONARQUBE_TOKEN = credentials('jenkins-sonarqube-token')
            }
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh '''
                    . venv/bin/activate
                    ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=aws_jenkins_1 \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://16.16.25.203:9000 \
                        -Dsonar.python.version=3.8 \
                        -Dsonar.login=${SONARQUBE_TOKEN} \
                        -Dsonar.ce.taskId= \
                        -Dsonar.ce.task.timeout=600s  # Set timeout for SonarQube task
                    '''
                }
                script {
                    waitForQualityGate(abortPipeline: false, credentialsId: 'jenkins-sonarqube-token')
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
