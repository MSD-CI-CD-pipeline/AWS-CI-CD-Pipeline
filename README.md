# AWS-CI-CD-Pipeline
Jenkins와 GitHub 웹훅을 활용한 CI/CD 파이프라인을 구축하였으며, 빌드된 JAR 파일을 AWS S3에 자동으로 업로드하여 배포 과정을 효율화하였습니다.

# ??

1. jenkins & github 에 웹 훅 걸기 
2. ngrok 
3. SpringBoot + DB 연동
4. S3 명령어 기반으로 순수 linux pipeline 으로 
    1. jenkins 에서 s3로 build 된 jar 업로드시 aws  cli  
5. S3에 jar 업로드 (최종) 
6. EC2 에서 서비스 구동 

1. jenkins 실행 
2. ngrok 등록 (ngrok http 8080)
3. jenkins 파이프라인 설정 

# 트러블 슈팅

## AWS CLI 설정

Jenkins 서버에 AWS CLI가 설치되어 있지 않거나 `PATH`에 포함되어 있지 않아서 발생하는 오류 발생 

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/75620ae2-9ad6-409a-a317-5ea81d4349ba/113129ec-9317-481d-ad5f-fcd9c14ccf11/image.png)

해결 → jenkins 서버에 aws cli 설치 

jenkins 컨테이너 안에서 sudo 명령어 쓸 수가ㅏ 없어서 root 계정으로 접속

```bash
docker exec -itu0 myjenkins bash 

sudo apt-get update

aws configure

```

```bash
pipeline {
    agent any
    environment {
        S3_BUCKET = 'ce35-bucket-02'
        AWS_SHARED_CREDENTIALS_FILE = '/root/.aws/credentials' // 자격 증명 파일 경로
        AWS_DEFAULT_REGION = 'ap-northeast-2' // 기본 리전 설정
    }
    stages {
        stage('Set AWS Region') {
            steps {
                script {
                    echo "Using AWS Region: ${env.AWS_DEFAULT_REGION}"
                }
            }
        }

        stage('Verify AWS CLI') {
            steps {
                script {
                    sh "aws --version"
                }
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/MSD-CI-CD-pipeline/AWS-CI-CD-Pipeline.git'
            }
        }

        stage('Build') {
            steps {
                dir('./step18_empApp') {
                    sh 'chmod +x gradlew'
                    sh './gradlew clean build -x test'
                    echo "Workspace: ${env.WORKSPACE}"
                }
            }
        }
        
        stage('Configure S3 Bucket') {
            steps {
                script {
		                sh "aws configure get region"
                    // 퍼블릭 액세스 차단 설정 제거
                    sh "aws s3api delete-public-access-block --bucket ${S3_BUCKET} --region ${AWS_DEFAULT_REGION}"
                    
                    // 버킷 소유권 제어 설정
                    sh "aws s3api put-bucket-ownership-controls --bucket ${S3_BUCKET} --ownership-controls '{\"Rules\":[{\"ObjectOwnership\":\"ObjectWriter\"}]}' --region ${AWS_DEFAULT_REGION}"
                    
                    // 버킷 ACL을 public-read로 설정
                    sh "aws s3api put-bucket-acl --bucket ${S3_BUCKET} --acl public-read --region ${AWS_DEFAULT_REGION}"
                }
            }
        }
        
        stage('Upload JAR to S3') {
            steps {
                script {
                    def jarFile = 'step18_empApp/build/libs/step18_empApp-0.0.1-SNAPSHOT.jar'
                    def s3Path = "s3://${S3_BUCKET}/"
                    sh "aws s3 cp ${jarFile} ${s3Path} --acl public-read --region ${AWS_DEFAULT_REGION}"
                }
            }
        }
    }
}

```

credentials 에 읽기 권한 추가 

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/75620ae2-9ad6-409a-a317-5ea81d4349ba/28534236-7748-44b5-9086-2a0883a48315/image.png)

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/75620ae2-9ad6-409a-a317-5ea81d4349ba/c7bfbcf6-fe63-465e-87d7-4ec995977f4c/image.png)

```bash
pipeline {
    agent any
    environment {
        S3_BUCKET = 'ce35-bucket-02'
        AWS_SHARED_CREDENTIALS_FILE = '/var/lib/jenkins/.aws/credentials' // 수정된 경로
        AWS_DEFAULT_REGION = 'ap-northeast-2'
    }
    stages {
        stage('Set AWS Region') {
            steps {
                script {
                    echo "Using AWS Region: ${env.AWS_DEFAULT_REGION}"
                }
            }
        }

        stage('Verify AWS CLI') {
            steps {
                script {
                    sh "aws --version"
                }
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/MSD-CI-CD-pipeline/AWS-CI-CD-Pipeline.git'
            }
        }

        stage('Build') {
            steps {
                dir('./step18_empApp') {
                    sh 'chmod +x gradlew'
                    sh './gradlew clean build -x test'
                    echo "Workspace: ${env.WORKSPACE}"
                }
            }
        }

        stage('Upload JAR to S3') {
            steps {
                script {
                    
                    def jarFile = 'step18_empApp/build/libs/step18_empApp-0.0.1-SNAPSHOT.jar'
                    def s3Path = "s3://${S3_BUCKET}/"
                    sh "aws s3 cp ${jarFile} ${s3Path} --acl public-read --region ${AWS_DEFAULT_REGION}"
                }
            }
        }
    }
}

```
