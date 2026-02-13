# JuiceFS CSI Driver

JuiceFS CSI Driver를 Helm으로 설치합니다.

- Metadata Engine: PostgreSQL (`postgres-rw.sandbox.svc.cluster.local`)
- Data Store: MinIO (`minio.sandbox.svc.cluster.local`)

## 구조

```
helm/juicefs-csi/
└── phase/
    └── sandbox/
        ├── values.yaml          # StorageClass 정의 (시크릿 제외)
        └── values-secret.yaml   # metaurl, accessKey, secretKey 포함
```

## 사전 준비

PostgreSQL과 MinIO가 먼저 배포되어 있어야 합니다.

```bash
# 1. PostgreSQL에 juicefs DB 생성
ssh sandbox001 "kubectl exec -n sandbox postgres-1 -- \
  psql -U postgres -c \"CREATE DATABASE juicefs OWNER sandbox;\""

# 2. MinIO에 juicefs 버킷 생성
#    MinIO Console (https://minio.revelly.io) 에서 생성

# 3. JuiceFS 파일시스템 포맷 (최초 1회)
ssh sandbox001 "kubectl run juicefs-format --rm -it --restart=Never \
  --image=juicedata/mount:latest -n sandbox -- \
  juicefs format \
  'postgres://<user>:<password>@postgres-rw.sandbox.svc.cluster.local:5432/juicefs?sslmode=disable' \
  juicefs-sandbox"
```

## 설치

```bash
helm repo add juicefs https://juicedata.github.io/charts/
helm repo update
ssh sandbox001 "KUBECONFIG=/etc/rancher/k3s/k3s.yaml helm install juicefs-csi juicefs/juicefs-csi-driver \
  -n kube-system -f -" < helm/juicefs-csi/phase/sandbox/values-secret.yaml
```

## StorageClass

| StorageClass | ReclaimPolicy | SubDir 패턴 | 용도 |
|-------------|---------------|-------------|------|
| `juicefs` | Delete | UUID (자동) | 임시 데이터, 삭제해도 되는 볼륨 |
| `juicefs-retain` | Retain | `{namespace}/{pvc-name}` | 영구 데이터, 예측 가능한 경로 |

## 사용 예시

```yaml
# 동적 프로비저닝
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-data
  namespace: sandbox
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: juicefs-retain
  resources:
    requests:
      storage: 10Gi
# → JuiceFS 내 경로: /sandbox/my-data
```

## 참고

- `juicefs-retain`의 `pathPattern`으로 PVC 삭제 후에도 데이터가 예측 가능한 경로에 남음
- 정적 프로비저닝(Static PV)으로 기존 경로를 `subPath`로 마운트 가능
- AccessMode: `ReadWriteMany` 지원 (여러 Pod 동시 마운트)
