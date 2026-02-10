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
        ├── patch-deployment.yaml   # URL, 리소스 하향
        └── patch-pvc.yaml          # storageClass: local-path
```

## 배포

```bash
# Raspberry Pi
kustomize build helm/ghost/overlays/pi/ | kubectl --context pi apply -f -

# 매니페스트 미리보기
kustomize build helm/ghost/overlays/pi/
```

## 접속

```bash
kubectl --context pi port-forward svc/ghost -n ghost 2368:2368
# http://localhost:2368
# Admin: http://localhost:2368/ghost
```
