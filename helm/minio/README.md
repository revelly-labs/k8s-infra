# MinIO

S3 호환 오브젝트 스토리지. JuiceFS의 데이터 백엔드로 사용합니다.

## 배포

```bash
# Sandbox (홈서버)
helm install minio minio/minio -n sandbox \
  -f helm/minio/phase/sandbox/values.yaml \
  -f helm/minio/phase/sandbox/values-secret.yaml
```

## 접속

- Console: https://minio.revelly.io
- API: https://minio-api.revelly.io
- 계정: values-secret.yaml 참조

## 참고

- Standalone 모드 (replicas: 1)
- StorageClass: `local-path` (10Gi)
- Ingress: Traefik 경유 (ClusterIP)
- `juicefs` 버킷이 JuiceFS 데이터 저장소로 사용됨
