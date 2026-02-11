# Ghost Blog

Ghost 블로그를 Kustomize로 배포합니다 (Helm chart 없음, SQLite 사용).

## 구조

```
helm/ghost/
├── base/
│   ├── kustomization.yaml    # Base
│   ├── deployment.yaml       # Deployment + PVC
│   └── service.yaml          # ClusterIP Service
└── overlays/
    └── pi/
        ├── kustomization.yaml      # Pi overlay
        ├── patch-deployment.yaml   # URL(revelly.io), 리소스 설정
        ├── patch-pvc.yaml          # storageClass: local-path
        └── patch-service.yaml      # NodePort 30080
```

## 배포

```bash
# Raspberry Pi
kustomize build helm/ghost/overlays/pi/ | kubectl --context pi apply -f -

# 매니페스트 미리보기
kustomize build helm/ghost/overlays/pi/
```

## 접속

- 외부: https://revelly.io (Cloudflare Tunnel → NodePort 30080)
- 로컬: http://192.168.0.5:30080
- Admin: https://revelly.io/ghost
