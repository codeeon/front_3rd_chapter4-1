# 프론트엔드 배포 파이프라인

![deploy-pipeline](https://github.com/user-attachments/assets/a9183bb8-5f77-4e69-9ca7-4b60b36c5838)

## 개요

GitHub Actions를 활용하여 프론트엔드 애플리케이션을 S3와 CloudFront로 배포하는 워크플로우를 작성합니다. 이 배포 파이프라인은 다음 단계를 포함합니다:

1. **저장소를 체크아웃**: 최신 코드를 GitHub에서 가져옵니다.
2. **Node.js 설정**: Node.js 18.x 버전을 사용하여 프로젝트 환경을 설정합니다.
3. **프로젝트 의존성 설치**: `npm ci` 명령어를 사용해 의존성을 설치합니다.
4. **Next.js 프로젝트 빌드**: `npm run build`를 실행하여 프로젝트를 빌드합니다.
5. **AWS 자격 증명 구성**: AWS 액세스 키와 비밀 키를 사용해 AWS 자격 증명을 설정합니다.
6. **S3에 배포**: 빌드된 파일을 S3 버킷에 동기화합니다.
7. **CloudFront 캐시 무효화**: 새로운 배포 내용을 적용하기 위해 CloudFront 캐시를 무효화합니다.

## 주요 링크

- [S3 버킷 웹사이트](http://codeeon-bucket-01.s3-website.ap-northeast-2.amazonaws.com)
- [CloudFront 배포](https://d12hv5oy8t4x9a.cloudfront.net)

## 주요 개념

### 1. GitHub Actions과 CI/CD 도구
GitHub Actions는 자동화된 워크플로우를 작성할 수 있는 CI/CD 도구입니다. 이를 통해 프로젝트 빌드와 배포를 자동화할 수 있습니다.

### 2. S3와 스토리지
AWS S3(Simple Storage Service)는 정적 웹사이트 호스팅을 지원하며 빌드된 정적 파일을 저장하고 배포하는 데 사용됩니다.

### 3. CloudFront와 CDN
AWS CloudFront는 콘텐츠 전송 네트워크(CDN)로, S3에 저장된 파일을 글로벌 사용자에게 빠르게 전달하기 위해 사용됩니다.

### 4. 캐시 무효화(Cache Invalidation)
CloudFront의 캐시는 배포된 콘텐츠를 빠르게 제공하지만, 업데이트된 파일이 반영되도록 캐시를 무효화해야 합니다.

### 5. Repository secret과 환경변수
배포 과정에서 민감한 정보(AWS 액세스 키, S3 버킷 이름 등)를 보호하기 위해 GitHub Secrets를 사용하여 환경변수를 관리합니다.

## 워크플로우 파일

```yaml
name: Next.js를 S3에 배포하고 CloudFront 무효화

on:
  push:
    branches:
      - main # 또는 master, 프로젝트의 기본 브랜치 이름에 맞게 조정
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: 리포지토리 체크아웃
        uses: actions/checkout@v2

      - name: 의존성 설치
        run: npm ci

      - name: 빌드
        run: npm run build

      - name: AWS 자격 증명 구성
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: S3에 배포
        run: |
          aws s3 sync out/ s3://${{ secrets.S3_BUCKET_NAME }} --delete

      - name: CloudFront 캐시 무효화
        run: |
          aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} --paths "/*"
```

## CDN과 성능 최적화

CloudFront를 사용하면 정적 파일의 전송 속도가 크게 향상되며, 이를 통해 사용자 경험을 개선할 수 있습니다. 추가적인 성능 개선을 위해 캐싱 전략과 Gzip 압축을 적용할 수 있습니다.

### S3
<img alt="스크린샷 2024-11-22 오전 10 54 29" src="https://github.com/user-attachments/assets/fcb8e6d9-52da-42e6-bde9-8c6c4d46bf99">

### CloudFront
<img alt="스크린샷 2024-11-22 오전 10 54 32" src="https://github.com/user-attachments/assets/e0107213-aeda-4b92-b53f-8c8c72037192">
