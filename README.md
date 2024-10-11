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
![11](https://github.com/user-attachments/assets/11f23328-4d5e-4b96-a496-d2565cae0b53)


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

![12](https://github.com/user-attachments/assets/17265561-14ea-4383-863f-4369215c710a)
![13](https://github.com/user-attachments/assets/736d988e-c9ef-4fd7-9d34-a61dc78c174b)


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

### EC2로 배포하기

### 1. Jenkins 컨테이너에서 PEM 키 설정

```bash
# 1. Jenkins 컨테이너에 접속
docker exec -itu0 myjenkins bash

# 2. .ssh 디렉토리 생성 및 권한 설정
root@617d3d46be32:/var/lib/jenkins# mkdir .ssh
root@617d3d46be32:/var/lib/jenkins# chown jenkins:jenkins .ssh
root@617d3d46be32:/var/lib/jenkins# chmod 700 .ssh

# 3. PEM 키 복사하기
username@servername:~$ docker cp ./ce35-key.pem myjenkins:/var/lib/jenkins/.ssh/ce35-key.pem

# 4. PEM 키 권한 설정
username@servername:~$ docker exec -u root myjenkins bash -c "chmod 400 /var/lib/jenkins/.ssh/ce35-key.pem"

```

### 2. EC2 인스턴스에 AWS CLI 설치 및 IAM 역할 부여

1. IAM 역할 생성
    - AWS Management Console에서 IAM으로 이동.
    - 역할 생성을 선택하고, AWS Service에서 EC2를 선택
    - 적절한 정책을 선택하여 EC2 인스턴스와 연결

### 3. Jenkins 파이프라인 스크립트

```groovy
pipeline {
    agent any
    environment {
        S3_BUCKET = 'ce35-bucket-02'
        AWS_SHARED_CREDENTIALS_FILE = '/var/lib/jenkins/.aws/credentials'
        AWS_DEFAULT_REGION = 'ap-northeast-2'
        EC2_INSTANCE_IP = '3.36.119.163'
        EC2_USER = 'ubuntu'
        JAR_FILE_NAME = 'step18_empApp-0.0.1-SNAPSHOT.jar'
        PEM_KEY_PATH = '/var/lib/jenkins/.ssh/ce35-key.pem' // PEM 키 경로
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
                git branch: 'main', url: '<https://github.com/MSD-CI-CD-pipeline/AWS-CI-CD-Pipeline.git>'
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
                    def jarFile = "step18_empApp/build/libs/${env.JAR_FILE_NAME}"
                    def s3Path = "s3://${S3_BUCKET}/step18_empApp/"
                    sh "aws s3 cp ${jarFile} ${s3Path} --acl public-read --region ${AWS_DEFAULT_REGION}"
                }
            }
        }

        stage('Copy JAR to EC2') {
            steps {
                script {
                    // EC2 인스턴스에서 S3에서 JAR 파일 다운로드
                    def copyCommand = """
                    ssh -i ${env.PEM_KEY_PATH} -o StrictHostKeyChecking=no ${env.EC2_USER}@${env.EC2_INSTANCE_IP} '
                    aws s3 cp s3://${S3_BUCKET}/step18_empApp/${env.JAR_FILE_NAME} /tmp/${env.JAR_FILE_NAME}
                    '
                    """
                    def result = sh(script: copyCommand, returnStatus: true)

                    if (result == 0) {
                        echo "Successfully copied JAR file to EC2 at /tmp/${env.JAR_FILE_NAME}"
                    } else {
                        error "Failed to copy JAR file to EC2. Command exited with status ${result}"
                    }
                }
            }
        }

        stage('Deploy and Run JAR on EC2') {
            steps {
                script {
                    // EC2 인스턴스에서 JAR 실행
                    def deployCommand = """
                    ssh -i ${env.PEM_KEY_PATH} -o StrictHostKeyChecking=no ${env.EC2_USER}@${env.EC2_INSTANCE_IP} '
                    chmod +x /tmp/${env.JAR_FILE_NAME} &&
                    sudo java -jar /tmp/${env.JAR_FILE_NAME}
                    '
                    """
                    def result = sh(script: deployCommand, returnStatus: true) // 실행 상태 반환
                    if (result == 0) {
                        echo "Successfully deployed and ran JAR file on EC2."
                    } else {
                        error "Failed to deploy and run JAR file on EC2. Command exited with status ${result}"
                    }
                }
            }
        }
    }
}

```

### 요약

- 위의 Jenkins 파이프라인은 S3에서 JAR 파일을 EC2로 복사한 후 실행하는 과정이다.
- PEM 키는 Jenkins 컨테이너 내의 `.ssh` 디렉토리에 저장되어 있으며, IAM 역할이 EC2 인스턴스에 적절하게 부여되어 있어야 한다.
- 스크립트 실행 후, JAR 파일이 EC2에서 성공적으로 배포되고 실행된다.
