# 🧪 Jenkins CI/CD 파이프라인 예제 !!!

이 프로젝트는 Jenkins를 사용하여 GitHub 소스코드를 기반으로 **빌드 → 테스트 → SonarQube 분석 → Docker 이미지 생성 및 배포**까지의 파이프라인을 자동화합니다.

---

## 📌 1. Jenkins Pipeline

```groovy
import java.text.SimpleDateFormat

def TODAY = (new SimpleDateFormat("yyMMddHHmm")).format(new Date())

pipeline {
    agent any
    environment {
        strDockerTag = "${TODAY}_${BUILD_ID}"
        strDockerImage = "sinsin09022/cicd_guestbook:${strDockerTag}"
    }
    stages {
        stage('Checkout'){
            steps{
                git branch: 'master',
                    url : 'https://github.com/haetsalshin92/guestbook.git'
            }
        }
        stage('build'){
            steps{
                sh "./mvnw clean package"
            }
        }
        stage('Unit Test'){
            steps{
                sh './mvnw test'
            }
            post{
                always{
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
            }
        }
        stage('SonarQube Analysis'){
            steps{
                withSonarQubeEnv('SonarQube-Server'){
                    sh '''
                        ./mvnw sonar:sonar \
                         -Dsonar.projectKey=guestbook \
                         -Dsonar.host.url=http://43.203.33.31:9000 \
                         -Dsonar.login=<토큰값>
                    '''
                }
            }
        }
        stage('SonarQube Quality Gate'){
            steps{
                timeout(time: 1, unit: 'MINUTES'){
                    script{
                        def qg = waitForQualityGate()
                        if(qg.status != 'OK'){
                            echo "NOT OK Status: ${qg.status}"
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        } else{
                            echo "OK Status: ${qg.status}"
                        }
                    }
                }
            }
        }
        stage('Docker image Build'){
            steps{
                script{
                    oDockImage = docker.build(strDockerImage, "--build-arg VERSION=${strDockerTag} -f Dockerfile .")
                }
            }
        }
        stage('Docker Image Push'){
            steps{
                script {
                    docker.withRegistry('', 'DockerHub_Credential'){
                        oDockImage.push()
                    }
                }
            }
        }
        stage('SSH Staging Server'){
            steps{
                sshagent(credentials: ['Staging-PrivateKey']){
                    sh "ssh -o StrictHostKeyChecking=no root@172.31.0.110 docker container rm -f guestbookapp"
                    sh "ssh -o StrictHostKeyChecking=no root@172.31.0.110 docker container run \
                        -d \
                        -p 38080:80 \
                        --name=guestbookapp \
                        -e MYSQL_IP=172.31.0.100 \
                        -e MYSQL_PORT=3306 \
                        -e MYSQL_DATABASE=guestbook \
                        -e MYSQL_USER=root \
                        -e MYSQL_PASSWORD=education \
                        ${strDockerImage}"
                }
            }
        }
    }
    post {
        always {
            slackSend(tokenCredentialId: 'slack-token',
                      channel: '#소셜',
                      color: 'good',
                      message: "${JOB_NAME} (${BUILD_NUMBER}) 빌드가 끝났습니다. Details: (<${BUILD_URL} | here>)")
        }
        success {
            slackSend(tokenCredentialId: 'slack-token',
                      channel: '#소셜',
                      color: 'good',
                      message: "${JOB_NAME} (${BUILD_NUMBER}) 빌드가 성공적으로 끝났습니다. Details: (<${BUILD_URL} | here>)")
        }
        failure {
            slackSend(tokenCredentialId: 'slack-token',
                      channel: '#소셜',
                      color: 'danger',
                      message: "${JOB_NAME} (${BUILD_NUMBER}) 빌드가 실패하였습니다. Details: (<${BUILD_URL} | here>)")
        }
    }
}
```

## 📌 2. dockerHub
![image](https://github.com/user-attachments/assets/2cfa7dcf-ea1b-4e73-9fcf-55ae91cb46d7)

## 📌 3. sonarQube
![image](https://github.com/user-attachments/assets/70c1114a-73f8-4ba1-8cac-101744fce17c)

## 📌 4. slack
![image](https://github.com/user-attachments/assets/62aba5ab-c416-4d93-8dc6-73af46a031c8)

## 📌 5. webhook
-> push가 이루어지면 자동으로 jenkins 빌드 시작
![image](https://github.com/user-attachments/assets/ab9e7219-0801-4790-9fa0-78766631709d)
