# JuiceFS S3 Gateway

JuiceFS 파일시스템을 S3 API로 노출하는 Gateway를 배포합니다.

## 구조

```
kustomize/juicefs-gateway/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml        # juicedata/mount:latest
│   └── service.yaml           # ClusterIP :9000
└── overlays/
    └── sandbox/
        ├── kustomization.yaml
        ├── secret.yaml         # METAURL, 인증 정보
        └── ingress.yaml        # juicefs.revelly.io → :9000
```

## 사전 준비

JuiceFS CSI Driver가 설치되고 파일시스템이 포맷되어 있어야 합니다.

## 배포

```bash
kubectl kustomize kustomize/juicefs-gateway/overlays/sandbox | ssh sandbox001 "kubectl apply -f -"
```

## 접속

- S3 API: https://juicefs.revelly.io
- 자격증명: `secret.yaml` 참조

## S3 클라이언트 접속 예시

```bash
# AWS CLI
aws configure
# Access Key / Secret Key: secret.yaml 참조
# Region: us-east-1

aws --endpoint-url https://juicefs.revelly.io s3 ls
```

## 참고

- JuiceFS 파일시스템의 전체 내용을 S3 API로 탐색/다운로드 가능
- MinIO와 별개: MinIO는 오브젝트 스토리지 백엔드, Gateway는 JuiceFS 뷰
