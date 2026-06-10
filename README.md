<h1>AWS 기반 글로벌 여행 후기 웹 서비스</h1>
<div align="center">

  <img src="https://github.com/user-attachments/assets/eba2c0a1-d226-4ba7-b410-ee1e08a91ac6" width="50%" alt="ViaGo Logo" />

  <br><br>

  <h3>✍️ 여행 기록하기 (사용자 페이지)</h3>
  <p>글로벌 사용자가 직관적으로 여행 리뷰, 별점, 사진 및 영상을 업로드할 수 있는 메인 UI입니다.</p>
  <img src="https://github.com/user-attachments/assets/7c8db257-b581-4b15-8f15-8eab9dcfc6b5" width="65%" alt="Travel Post Form" />

  <br><br>
  <hr style="border: 1px dashed #e1e4e6;" />
  <br>

  <h3>🤖 AI 기반 게시글 검토 대시보드 (관리자 페이지)</h3>
  <p>Amazon Rekognition과 Comprehend가 감지한 유해성 점수 및 필터링 사유를 실시간으로 모니터링하고 관리하는 공간입니다.</p>
  <img src="https://github.com/user-attachments/assets/78421846-9210-41b9-b924-dedef03417f2" width="85%" alt="Admin Dashboard" />

</div>
<h2>AWS 구성도</h2>
<img width="1007" height="879" alt="image" src="https://github.com/user-attachments/assets/04be2de1-8d2c-49c4-97b8-b92e185d1969" />




## 🌐 End-to-End 글로벌 트래픽 처리 흐름

글로벌 사용자가 웹 서비스에 접속하여 여행 후기(텍스트 및 이미지)를 업로드하고 시스템이 이를 처리하는 전체 아키텍처 흐름은 다음과 같습니다.

### 1단계: 글로벌 라우팅 및 네트워크 가속 (Edge Layer)
* **Route 53 (지리적 라우팅):** 글로벌 사용자가 도메인에 접속하면, `Route 53`이 사용자의 IP 위치를 분석하여 가장 가까운 리전(Seoul 또는 São Paulo)의 엔드포인트로 트래픽을 유도합니다.
* **Global Accelerator & CloudFront:** 전 세계 Edge Location을 통해 네트워크 지연을 최소화하고, 정적 콘텐츠(웹 에셋, 기존 이미지 등)는 `CloudFront` 캐싱을 통해 백엔드 서버를 거치지 않고 고속으로 즉시 반환합니다.

### 2단계: 웹 보안 검증 및 부하 분산 (Ingress Layer)
* **AWS WAF (Web Application Firewall):** 외부의 악의적인 공격(SQL Injection, Cross-Site Scripting, DDoS 등)을 `WAF`가 최전방에서 탐지하고 차단하여 인프라 안전성을 확보합니다.
* **ACM (Certificate Manager) & ALB:** `ACM`으로 발급받은 SSL/TLS 인증서를 기반으로 안전한 HTTPS 암호화 통신을 수행하며, `ALB (Application Load Balancer)`가 현재 구동 중인 EC2 인스턴스들에게 트래픽을 균등하게 분산합니다.

### 3단계: 애플리케이션 처리 및 동적 확장 (Compute Layer)
* **EC2 & Auto Scaling Group:** 서비스의 핵심 로직이 실행되는 구간입니다. 이벤트나 시간대별 트래픽 급증 시 `Auto Scaling` 정책에 따라 EC2 인스턴스가 자동으로 증설(Scale-Out)되어 서비스 중단 없는 고가용성을 유지합니다.
* **RDS MariaDB:** 사용자 정보, 여행 후기 텍스트 데이터 등 정형 데이터는 가용 영역(AZ)이 분리된 내부망(Private Subnet)의 `RDS`에 안전하게 저장 및 관리됩니다.

### 4단계: AI 기반 미디어 업로드 및 유해 콘텐츠 검열 파이프라인
* **S3 Bucket (정적 파일 스토리지):** 사용자가 새로 업로드한 여행지 사진이나 영상은 가용성과 내구성이 뛰어난 `S3` 버킷에 원본 데이터로 격리 저장됩니다.
* **Amazon Rekognition (이미지 검열):** 이미지 업로드 이벤트가 발생하면 `Rekognition` API가 호출되어 해당 사진 내의 성인물, 노출, 폭력성 등 부적절한 요소(Moderation Labels)를 자동 분석 및 판별합니다.
* **Amazon Comprehend (텍스트 검열):** 후기 본문 텍스트는 `Comprehend` 자연어 처리(NLP)를 거쳐 무분별한 광고성 스팸 키워드, 악성 댓글, 개인정보 유출 여부를 실시간으로 감지합니다.
* **결과 처리:** AI 분석 결과 '정상'으로 판별된 콘텐츠만 서비스 피드에 노출되며, 유해 콘텐츠로 분류될 경우 자동으로 블라인드 처리 및 관리자 페이지에 전송됩니다.

### 5단계: 시스템 관리 및 모니터링 (Operations Layer)
* **OpenVPN망 접속:** 인프라 관리자는 외부 공인망이 아닌, `OpenVPN` 게이트웨이를 통해서만 내부망(Private Subnet)의 EC2 및 데이터베이스 서버에 `MobaXTerm`이나 `PuTTY`로 안전하게 원격 접속할 수 있습니다.
* **CloudWatch & Grafana:** 시스템에서 발생하는 모든 로그와 메트릭(CPU, Traffic, 인프라 상태)은 `CloudWatch`로 수집되며, 이를 `Grafana` 대시보드와 연동하여 실시간 시각화 및 장애 전파 모니터링 환경을 제공합니다.


## 🛠️ Tech Stack & AWS 서비스 상세

이 프로젝트에 사용된 기술 스택과 AWS 핵심 자원들의 역할 분담은 다음과 같습니다.

| 분류 | 기술 Stack / 서비스 아이콘 | 역할 및 활용 목적 |
| :--- | :--- | :--- |
| **Traffic & CDN** | `Route 53` `CloudFront` <br> `Global Accelerator` | - 글로벌 사용자의 전송 지연 시간(Latency) 최소화<br>- Edge Location을 활용한 정적 콘텐츠 캐싱 및 라우팅 최적화 |
| **Compute & Network** | `VPC` `Internet Gateway` `NAT Gateway`<br>`ALB` `EC2` `Auto Scaling` | - 외부망(Public)과 내부망(Private) 구분을 통한 가용 구역 설계<br>- 부하 분산 및 트래픽 증가 시 자동으로 서버를 확장하는 고가용성 환경 구축 |
| **Storage & DB** | `S3 Bucket` | - 여행 후기에 업로드되는 대용량 이미지 및 미디어 파일의 안정적인 저장소 활용 |
| **Security & Auth** | `IAM` `WAF` `Certificate Manager`<br>`OpenVPN` | - ACM을 통한 SSL/TLS 암호화 적용<br>- WAF를 통한 웹 취약점 방어 및 OpenVPN을 이용한 안전한 관리자 접속 제어 |
| **AI Resource** | `Comprehend` `Rekognition` | - 업로드된 여행지 리뷰 텍스트 및 이미지 분석<br>- 부적절한 광고성 글이나 유해 이미지(음란물 등) 자동 검열 파이프라인 구현 |
| **DevOps & Tools** | `CloudFormation` `CloudWatch` `Grafana`<br>`MobaXTerm` `PuTTYgen` | - IaC(Code 기반 인프라)를 통한 멀티 리전 아키텍처 배포 자동화<br>- 실시간 자원 사용량 모니터링 및 안전한 서버 원격 제어 환경 구축 |
| **Productivity** | `Notion` `ChatGPT` `VS Code` `draw.io` | - 프로젝트 문서화, 아키텍처 설계, 인프라 코드 작성 및 협업 |

사용된 자원

<p align="center">
  <img src="https://github.com/user-attachments/assets/c9179869-7a99-4b6b-b791-f3adb0e3bbc9" width="45%">
  <img src="https://github.com/user-attachments/assets/ddddc982-c156-4abb-91ef-af08c4142d42" width="45%" align="top">
</p>


## 🤖 CloudFormation을 활용한 인프라 자동화 (IaC)

이 프로젝트는 복잡한 멀티 리전(Seoul & São Paulo) 인프라를 수동으로 구축하지 않고, **AWS CloudFormation**을 통해 코드로 관리(Infrastructure as Code)하여 휴먼 에러를 방지하고 배포 효율성을 극대화했습니다.

### 1. IaC 도입으로 얻은 이점
* **멀티 리전 신속 배포:** 동일한 아키텍처 템플릿을 사용하여 서울 리전 구축 후, 상파울루 리전의 인프라를 단 수 분 만에 복제 및 배포 완료했습니다.
* **인프라 버전 관리:** 환경 설정의 변경 이력을 Git 커밋 로그로 추적하여, 문제 발생 시 즉각적인 롤백 및 트래픽 안정성을 확보했습니다.
* **휴먼 에러 제거:** 서브넷 CIDR 블록 배치, 라우팅 테이블 연결, 보안 그룹(Security Group) 규칙 설정을 자동화하여 수동 설정 시 발생할 수 있는 누락과 보안 구멍을 원천 차단했습니다.

## 🚀 Key Features & Architecture Highlights

이 프로젝트는 글로벌 확장성, 고가용성, 그리고 AI 기반의 자동화된 보안/검열 시스템을 목표로 다음과 같은 인프라 환경을 설계 및 구축했습니다.

* **🌐 글로벌 유저 최적화 멀티 리전 아키텍처**
  - `Route 53`의 지리적 라우팅 및 `Global Accelerator`를 연동하여 글로벌 사용자의 네트워크 지연 시간(Latency)을 최소화하고 리전 수준의 장애 조치(Failover) 환경을 마련했습니다.

* **📈 ALB & Auto Scaling 기반의 고가용성(HA) 확보**
  - 가용 영역(AZ) 전반에 걸쳐 트래픽을 분산하는 `ALB`와, 부하 상태에 따라 인스턴스를 동적으로 확장하는 `Auto Scaling Group`을 결합하여 무중단 서비스 환경을 구현했습니다.

* **🗄️ RDS MariaDB를 활용한 3-Tier 데이터 계층 분리**
  - 웹/애플리케이션 서버와 데이터베이스 계층을 물리·논리적으로 분리하고, `RDS`를 Private Subnet에 격리 배치하여 데이터의 가용성과 완벽한 보안성을 확보했습니다.

* **📦 S3 & CloudFront를 통한 미디어 전송 최적화**
  - 대용량 여행 후기 이미지와 영상 에셋을 `S3` 버킷에 안정적으로 저장하고, `CloudFront` CDN 캐싱 기술을 통해 전 세계 사용자에게 정적 콘텐츠를 고속으로 전송합니다.

* **🔒 OpenVPN 기반의 안전한 관리자 접속 환경망(Bastion)**
  - 공인망 노출을 최소화하기 위해 내부 인프라 관리자망을 분리하고, 오직 `OpenVPN` 게이트웨이를 통해서만 Private 자원에 보안 원격 접속(SSH/데이터베이스 관리)이 가능하도록 제어했습니다.

* **🛡️ ACM · WAF · Security Group의 다층 방어 체계**
  - `ACM`을 통한 HTTPS 암호화 통신 기본 적용, 웹 최전방에서 악성 페이로드를 차단하는 `AWS WAF`, 그리고 인스턴스별 프로토콜/포트를 최소화하는 `보안 그룹(Security Group)`을 통해 철저한 다층 보안을 실현했습니다.

* **🤖 CloudFormation 기반의 인프라 코드화 (IaC)**
  - 전체 멀티 리전 인프라 구성을 `CloudFormation` 템플릿(YAML)으로 자산화하여 휴먼 에러를 원천 차단하고, 동일한 프로덕션 환경을 단 수 분 만에 신속하게 복제·배포할 수 있도록 자동화했습니다.

* **👁️ AI(Comprehend, Rekognition) 연동 유해 콘텐츠 자동 검열 파이프라인**
  - 사용자가 업로드하는 리뷰 데이터의 신뢰성을 위해 `Rekognition`으로 이미지 내 음란물·폭력성을 실시간 검업하고, `Comprehend` NLP 분석을 통해 스팸 광고 및 유해 텍스트를 자동으로 블라인드 처리하는 스마트 거버넌스를 구축했습니다.
