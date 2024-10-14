# Jenkins 기반 CI/CD 파이프라인 구축 및 S3 자동 배포

<img width="1000" alt="2222" src="https://github.com/user-attachments/assets/15628949-d23e-49ab-9e9c-dcedf778bc74">


## 🚪 들어가기 전
효율적인 CI/CD 파이프라인을 구축하여 애플리케이션 배포를 자동화하는 것입니다. 이를 위해 다음과 같은 단계로 작업을 진행하였습니다.

`Jenkins와 GitHub에 웹 훅 설정`: Jenkins와 GitHub 간의 원활한 통신을 위해 웹 훅을 설정하여, 코드 변경 시 자동으로 빌드가 트리거되도록 하였습니다.

`Ngrok 설정`: 로컬 개발 환경에서 Jenkins 서버에 외부 접근을 가능하게 하기 위해 Ngrok을 활용하여 안전하게 터널링을 구성하였습니다.

`Spring Boot와 데이터베이스 연동`: Spring Boot 애플리케이션을 데이터베이스와 연결하여, 사용자 요청에 대한 데이터를 효과적으로 관리하고 처리할 수 있도록 하였습니다.

`순수 Linux 파이프라인으로 S3 명령어 기반 작업 진행`: Jenkins를 통해 AWS CLI를 활용하여 빌드된 JAR 파일을 S3에 자동으로 업로드하는 프로세스를 설정하였습니다. 이를 통해 수동 작업을 최소화하고 배포의 일관성을 높였습니다.

`S3에 최종 JAR 파일 업로드`: 빌드가 완료된 후, JAR 파일을 S3에 업로드하여 안정적인 저장소에 배포 아티팩트를 보관하였습니다.

`EC2에서 서비스 구동`: AWS EC2 인스턴스에서 Spring Boot 애플리케이션을 실행하여, 클라우드 환경에서의 서비스 제공을 최적화하였습니다.

이러한 과정은 애플리케이션의 배포 효율성을 높이고, 지속적인 통합 및 배포를 가능하게 하여 개발 및 운영 프로세스를 향상시키는 데 기여하였습니다.
<br/><br/>

## 👥 팀원 소개

| 노솔리 | 구동길 | 홍민영 |
|:-----------:|:-----------:|:-----------:|
| <img width="100px" src="https://avatars.githubusercontent.com/soljjang777" /> | <img width="100px" src="https://avatars.githubusercontent.com/dkac0012"/> | <img width="100px" src="https://avatars.githubusercontent.com/u/65701100?v=4"/> |
| [@soljjang777](https://github.com/soljjang777) | [@dkac0012](https://github.com/dkac0012) | [@HongMinYeong](https://github.com/HongMinYeong) |

<br/><br/>

## 💻 시스템 환경 및 소프트웨어

| **항목**            | **상세 정보**                                                                                  |
|---------------------|-----------------------------------------------------------------------------------------------|
| **운영 체제**       | <img src="https://img.shields.io/badge/Ubuntu 22.04.5 LTS-E95420?style=flat-square&logo=Ubuntu&logoColor=white"/>  |
| **클라우드 서비스** | <img src="https://img.shields.io/badge/amazonec2-FF9900?style=flat-square&logo=amazonec2&logoColor=white"/> <br> <img src="https://img.shields.io/badge/amazonrds-527FFF?style=flat-square&logo=amazonrds&logoColor=white"/>  <br><img src="https://img.shields.io/badge/amazons3-569A31?style=flat-square&logo=amazons3&logoColor=white"/>  |
| **CI/CD 도구**      | <img src="https://img.shields.io/badge/jenkins-D24939?style=flat-square&logo=jenkins&logoColor=white"/>  |
| **터미널 에뮬레이터** | <img src="https://img.shields.io/badge/MobaXterm-241F31?style=flat-square&logo=MobaXterm&logoColor=white"/>  |
| **컨테이너화**       | <img src="https://img.shields.io/badge/Docker 27.3.1-2496ED?style=flat-square&logo=Docker&logoColor=white"/>  |
| **가상화 소프트웨어** | <img src="https://img.shields.io/badge/VirtualBox-183A61?style=flat-square&logo=virtualbox&logoColor=white"/>  |
| **CLI 도구**          | <img src="https://img.shields.io/badge/AWS_CLI_v2-232F3E?style=flat-square&logo=amazonaws&logoColor=white"/>  |


<br/><br/>

## 🚀 주요 기능


### 1. 자동화된 배포 프로세스

- GitHub 저장소에 코드 푸시 시 자동 빌드 트리거
- 빌드 결과물(JAR 파일)을 AWS S3에 자동 업로드
- EC2 인스턴스에 최신 버전 애플리케이션 자동 배포


### 2. 데이터베이스 연동

- Amazon RDS MySQL과 연동하여 데이터 영속성 보장

### 3. AWS 보안 

- IAM 역할을 사용하여 EC2 인스턴스에 권한 부여

<br/><br/>

## ⛯ 자동화된 배포 과정 다이어그램 

 <img src="https://github.com/user-attachments/assets/e9eea955-49ca-4150-980b-375067e04bf2" width="75%">

<br/><br/>


## 1️⃣ 작업 1:Jenkins & Github 에 웹 훅 걸기 
1. `GitHub 웹 훅 설정`: Jenkins와 GitHub를 연동하기 위해, GitHub에서 웹 훅을 생성하여 Jenkins로의 트리거를 설정하였습니다.

2. `Ngrok을 이용한 Jenkins 외부 접근 설정`: 로컬 네트워크에서 구동 중인 Jenkins에 외부에서 접근할 수 있도록 Ngrok을 활용하여 안전한 터널링을 구성하였습니다. 이를 통해 GitHub 웹 훅이 로컬 Jenkins 서버에 접근할 수 있도록 했습니다.

3. `Jenkins에서 GitHub 리포지토리 연동`: Jenkins에서 GitHub 리포지토리를 등록하고, 빌드가 자동으로 트리거되도록 설정하였습니다.

<br/><br/>

## 2️⃣ 작업 2:Jenkins에서 S3로 JAR 파일 빌드 후 업로드
### 1. Jenkins 서버에 AWS CLI 설치
```bash
# Jenkins 컨테이너에서 sudo 명령어 사용이 제한되어, root 계정으로 전환하여 작업했습니다. 
username@myserver01:~$ docker exec -u root -it myjenkins  /bin/bash

root@b487084d5f73:/# apt-get update

root@b487084d5f73:/# apt-get install awscli -y

root@b487084d5f73:/# aws configure
AWS Access Key ID [****************JS2T]:
AWS Secret Access Key [****************qPca]:
Default region name [None]: ap-northeast-2
Default output format [None]: json
```
* 만약 해당 과정 생략 후 빌드 시 아래와 같은 오류 발생했습니다. (Jenkins 서버에 AWS CLI가 설치되어 있지 않거나 `PATH`에 포함되어 있지 않아서 발생하는 오류)
     
   <img src="https://github.com/user-attachments/assets/11f23328-4d5e-4b96-a496-d2565cae0b53" width="65%">

<br/>

### 2. Jenkins AWS 자격 증명에 읽기 권한 추가
* Jenkins에서 AWS 리소스(S3 등)에 안전하게 접근할 수 있도록, AWS 자격 증명에 필요한 읽기 권한을 추가했습니다.
```bash
# 읽기 권한 확인 (없음)
root@b487084d5f73:/# ls -l ~/.aws/credentials
-rw------- 1 root root 116 Oct 11 02:52 /root/.aws/credentials

# 읽기 권한 추가
root@b487084d5f73:/# chmod 640 /root/.aws/credentials

# 읽기 권한 확인
root@b487084d5f73:/# ls -l ~/.aws/credentials
-rw-r----- 1 root root 116 Oct 11 02:52 /root/.aws/credentials

root@b487084d5f73:/# mkdir -p /var/lib/jenkins/.aws

root@b487084d5f73:/var/lib# chmod 775 jenkins 

root@b487084d5f73:/# cp ~/.aws/credentials /var/lib/jenkins/.aws/

root@b487084d5f73:/# cat /var/lib/jenkins/.aws/credentials
[default]
aws_access_key_id = AK****************
aws_secret_access_key = QG***********************
```
<br/>

### 3. Jenkins에서 S3 자동 업로드 파이프라인 작성
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

<br/><br/>

## 3️⃣ 작업 3:EC2로의 배포

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
<br/>

### 2. EC2 인스턴스에 AWS CLI 설치 및 IAM 역할 부여

1. IAM 역할 생성
    - AWS Management Console에서 IAM으로 이동.
    - 역할 생성을 선택하고, AWS Service에서 EC2를 선택
    - 적절한 정책을 선택하여 EC2 인스턴스와 연결
<br>
1-1. 역할 생성 클릭
<br>
 <img src="https://github.com/user-attachments/assets/8e02168a-7e17-4a29-9026-a0da0a7a95ce" width="75%">

1-2. AWS Service 선택 - EC2
<br>
 <img src="https://github.com/user-attachments/assets/1d090d9f-e248-468d-8c97-a1a4a9c5a8f2" width="75%">

1-3. 역할 생성하고 해당 EC2 인스턴스와 연결 
<br>
 <img src="https://github.com/user-attachments/assets/5deafa5c-f4c2-4047-8023-893437f8afa1" width="75%">
 
 ```bash
|+--------------------------------------------------+--------------------------------+|||
||||                                IamInstanceProfile                                 ||||
|||+-------+---------------------------------------------------------------------------+|||
||||  Arn  |  arn:aws:iam::64********:instance-profile/ce35-user                     ||||
||||  Id   |  AIPAZ******************                                                  ||||
|||+-------+---------------------------------------------------------------------------+
```

### 3. Jenkins 파이프라인 스크립트
- S3에서 EC2로 복사하는 대신, EC2에 직접 접근하여 S3에 있는 JAR 파일을 다운로드하도록 수정합니다. 이 작업을 수행하기 위해서는 EC2 인스턴스에 적절한 IAM 권한을 부여해야 합니다. IAM 권한이 없으면 S3에서 파일을 다운로드할 수 없습니다.

```groovy
pipeline {
    agent any
    environment {
        S3_BUCKET = 'ce35-bucket-02'
        AWS_SHARED_CREDENTIALS_FILE = '/var/lib/jenkins/.aws/credentials'
        AWS_DEFAULT_REGION = 'ap-northeast-2'
        EC2_INSTANCE_IP = '3.3*.***.***'
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
                    ssh -i ${env.PEM_KEY_PATH} -o StrictHostKeyChecking=no ${env.EC2_USER}@${env.EC2_INSTANCE_IP}
                    aws s3 cp s3://${S3_BUCKET}/step18_empApp/${env.JAR_FILE_NAME} /tmp/${env.JAR_FILE_NAME}
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
                    ssh -i ${env.PEM_KEY_PATH} -o StrictHostKeyChecking=no ${env.EC2_USER}@${env.EC2_INSTANCE_IP}
                    chmod +x /tmp/${env.JAR_FILE_NAME} &&
                    sudo java -jar /tmp/${env.JAR_FILE_NAME}
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
<br/>

### EC2 배포 후 파일 구조 확인

```bash
ubuntu@ip-10-11-6-115:~$ tree
.
├── awscliv2.zip
└── step18_empApp-0.0.1-SNAPSHOT.jar
```

### 요약

- 위의 Jenkins 파이프라인은 S3에서 JAR 파일을 EC2로 복사한 후 실행하는 과정입니다.
- PEM 키는 Jenkins 컨테이너 내의 `.ssh` 디렉토리에 저장되어 있으며, IAM 역할이 EC2 인스턴스에 적절하게 부여되어 있어야 합니다.
- 스크립트 실행 후, JAR 파일이 EC2에서 성공적으로 배포되고 실행됩니다.

<br/><br/>

## 🛠 트러블슈팅

프로젝트 진행 중 다양한 문제에 직면했고, 이를 해결하는 과정에서 많은 것을 배웠다. 주요 트러블슈팅 사례는 다음과 같습니다:

1. **Jenkins 서버의 AWS CLI 설정 문제**
    - 문제: Jenkins 서버에서 AWS CLI 명령어 실행 시 권한 오류 발생
    - 원인: Jenkins 컨테이너 내 AWS CLI 미설치 및 IAM 자격 증명 미설정
    - 해결:
        1. Jenkins 컨테이너에 AWS CLI 설치
        2. IAM 사용자 생성 및 적절한 권한 부여
        3. AWS configure 명령어로 자격 증명 설정
    - 학습: 컨테이너 환경에서의 툴 설치 및 권한 관리의 중요성
2. **S3 버킷 접근 권한 문제**
    - 문제: Jenkins에서 S3로 파일 업로드 실패
    - 원인: S3 버킷의 퍼블릭 액세스 차단 설정 및 부적절한 버킷 정책
    - 해결:
        1. S3 버킷의 퍼블릭 액세스 차단 설정 제거
    - 학습: AWS 리소스의 세부적인 권한 설정 방법
3. **EC2 인스턴스로의 JAR 파일 배포 문제**
    - 문제: Jenkins에서 EC2로 JAR 파일 전송 실패
    - 원인: EC2 인스턴스의 보안 그룹 설정 및 SSH 키 관리 문제
    - 해결:
        1. Jenkins에 EC2 접속용 프라이빗 키 등록 및 권한 설정
        2. 배포 스크립트 내 SSH 명령어 수정
    - 학습: 클라우드 환경에서의 네트워크 보안 설정 및 안전한 키 관리의 중요성
