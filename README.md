# 프론트엔드 배포 파이프라인
- AWS 내에 S3, CloudFront, IAM을 생성하여 프로젝트 배포 파이프라인을 구성하였습니다.
- Next.js 프로젝트 생성 후, GitHub에 올리고 AWS와 GitHub Actions를 연동하였습니다.

## 배포 다이어그램
![Image](https://github.com/user-attachments/assets/c312a1c3-b3e6-4243-9d4a-8c169610188b)

## 주요 링크
- S3 버킷 웹사이트 엔드포인트 : http://hangae-s3-demo-bucket.s3-website-us-east-1.amazonaws.com
- CloudFrount 배포 도메인 이름 : https://dref3bg2xtuff.cloudfront.net

## 주요 개념
- GitHub Actions과 CI/CD 도구
  1. GitHub Actions 개념
     - GitHub에서 제공하는 CI/CD 자동화 도구
     - YAML 파일을 이용해 workflow를 정의하고, push/pull request/tag 등 발생하면 자동으로 실행
  2. CI/CD 개념
     - 소프트웨어 개발의 빌드, 테스트, 배포를 자동화
     - CI (Continuous Integration, 지속적 통합) : 코드 변경이 있을 때마다 자동으로 빌드 및 테스트 수행
     - CD (Continuous Deployment/Delivery, 지속적 배포/전달)
       - Continuous Delivery: 특정 환경에서 수동 승인 후 배포
       - Continuous Deployment: 모든 배포 과정을 자동화
      
  
  ** 과제에서 GitHub Actions를 이용해 AWS S3에 자동 배포함 **
  
- S3와 스토리지
  1. AWS S3(Simple Storage Service) 개념
     - 정적 웹사이트 배포, 백업, 데이터 아카이빙 등에 사용
     - S3 특징
       1. 무제한 스토리지: 데이터 크기 제한 없이 저장 가능
       2. 파일 단위 저장(Object Storage): 파일을 객체(Object)로 저장하며, 각 객체는 키(Key)와 메타데이터를 가짐
       3. 내구성 & 가용성: 99.999999999% (11 9s) 내구성 제공 (데이터 손실 거의 없음)
       4. 버킷 단위 관리: 파일을 저장할 컨테이너 역할 (ex. my-nextjs-bucket)
       5. 정적 웹사이트 호스팅 기능: 직접 S3 버킷에서 웹사이트를 서빙할 수도 있음
      - 버킷 : S3에서 데이터를 저장하는 기본 단위(=폴더), 고유한 이름 가지기
      - 객체 : S3에 저장된 개별 파일
  2. 스토리지(Storage) 개념
      - 데이터를 저장하는 공간을 의미하며, 사용 목적에 따라 여러 가지 유형 존재
      - 파일 스토리지(파일), 블록 스토리지(EBS, SSD), Object 스토리지(AWS S3)

 
  ** 과제에서 Object 스토리지를 활용 **
  
- CloudFront와 CDN
  1. AWS CloudFront 개념
     - AWS에서 제공하는 글로벌 CDN 서비스로, S3,EC2, API Gateway 등 연동하여 빠르고 안전하게 콘텐츠를 배포할 수 있음.
     - 엣지 서버(Edge Location) 사용하여 사용자와 가까운 곳에서 콘텐츠 제공
     - 정적/동적 콘텐츠 캐싱 지원 -> HTML, CSS, JS, 이미지, 폰트 등 캐싱 가능
     - DDoS 방어, AWS WAF(Web Application Firewall) 통합 가능
  2. CDN(Content Delivery Network) 개념
     - 전 세계에 분산된 서버를 활용하여 콘텐츠를 빠르게 제공하는 네트워크


- 캐시 무효화(Cache Invalidation)
  1. 캐시 무효화
     - CDN이나 브라우저에 저장된 기존 캐시를 삭제하거나 새로운 버전으로 교체하는 방법
     - 왜 캐시 무효화가 필요할까?
       - CDN이 캐싱한 정적 파일은 기본적으로 TTL이 만료될 때까지 유지됨. Next.js 정적 파일을 업데이트했을 때 기존 캐시가 유지되면 변경 사항이 즉시 반영되지 않음. 따라서 CloudFront 캐시를 강제로 삭제하여 최신 콘텐츠를 반영하는 과정이 필요
     - GitHub Actions에서 배포 후 캐시 무효화 방법
       ```
        - name: Invalidate CloudFront cache
          run: |
            aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} --paths "/*"
       ```
       
- IAM (Identity and Access Management)
  1. IAM 개념
     - AWS 리소스에 대한 접근을 제어하는 서비스
     - 사용자, 역할, 정책을 생성하고 권한 관리를 통해 보안성 강화
    

  ** 과제에서 Next.js를 AWS S3 & CloudFront에 배포할 때, GitHub Actions이 AWS에 배포할 수 있도록 IAM 사용자를 생성하고 Access Key & Secret Key를 발급받음 **


- Repository secret과 환경 변수
  1. Repository secret
     - GitHub Actions에서는 보안이 필요한 환경변수를 직접 코드에 저장하지 않고, Repository Secret을 사용하여 관리
     - Repository Secret은 GitHub의 암호화된 저장 공간에 보관되며, GitHub Actions 워크플로우에서만 접근 가능
     - AWS 배포를 위한 Secret 설정 항목
       - AWS_ACCESS_KEY_ID : AWS IAM 액세스 키
       - AWS_SECRET_ACCESS_KEY : AWS IAM 비밀 키
       - AWS_REGION : AWS 배포 지역 (예: us-east-1)
       - S3_BUCKET_NAME : 배포할 S3 버킷 이름
       - CLOUDFRONT_DISTRIBUTION_ID : CloudFront 배포 ID
  2. 환경 변수
     - 운영 체제 또는 애플리케이션이 실행될 때 필요한 설정 값을 저장하는 변수
     - Next.js, GitHub Actions, AWS CLI 등 다양한 개발 환경에서 API 키, 데이터베이스 정보, 서버 설정 값 등을 보관하는 용도로 사용


# CDN과 성능최적화
- 왼쪽은 S3를 통해 받은 응답 속도와 파일 사이즈이고 오른쪽은 CloudFront를 통해 받은 응답 속도와 파일 사이즈이다.
- 캡처 화면을 통해 CDN을 사용했을 때와 사용하지 않았을 때의 차이를 비교할 수 있다.

  
  ![Image](https://github.com/user-attachments/assets/27664700-c375-45c9-ac66-b6567d235e8c)
  | |파일 사이즈|응답 시간|
  |---|---:|---:|
  |S3|11.9kB|445ms|
  |CloudFront|3.1kB|60ms|

- S3에서 직접 제공하는 파일보다 CloudFront가 제공하는 파일 크기가 70% 정도 감소
- CloudFront 사용 시 응답 속도가 약 7배 정도 향상됨

- 성능 차이가 나는 원인
  - 응답 속도 차이
    - S3는 특정 AWS 리전(예: us-east-1)에 위치해 있으며, 사용자와 물리적으로 멀 경우 응답 시간이 길어짐
    - CloudFront는 전 세계 엣지 서버(Edge Location)에서 캐싱된 데이터를 제공하여 응답 시간이 단축됨
  - 캐싱(Caching) 효과
    - S3는 요청할 때마다 원본 데이터를 직접 제공 (응답 시간이 상대적으로 길어짐)
    - CloudFront는 한 번 요청된 데이터를 엣지 서버에 캐싱하여 재요청 시 더 빠르게 제공



