# CloudNativePG Operator

PostgreSQL을 위한 Kubernetes Operator. 클러스터당 1회 설치합니다.

## 설치

```bash
# sandbox001
helm repo add cloudnative-pg https://cloudnative-pg.github.io/charts
helm repo update
KUBECONFIG=/etc/rancher/k3s/k3s.yaml helm install cnpg cloudnative-pg/cloudnative-pg \
  -n cnpg-system --create-namespace
```

## 삭제

```bash
KUBECONFIG=/etc/rancher/k3s/k3s.yaml helm uninstall cnpg -n cnpg-system
```

## 참고

- Operator는 `cnpg-system` 네임스페이스에 설치
- 실제 PostgreSQL 클러스터 정의는 `kustomize/postgres/` 에서 관리
- CRD: `Cluster`, `Backup`, `ScheduledBackup` 등
