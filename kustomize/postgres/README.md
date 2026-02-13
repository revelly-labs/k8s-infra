# PostgreSQL (CloudNativePG)

CloudNativePG Operator로 PostgreSQL 클러스터를 배포합니다.

## 구조

```
kustomize/postgres/
├── base/
│   ├── kustomization.yaml
│   └── cluster.yaml            # CNPG Cluster 정의
└── overlays/
    └── sandbox/
        ├── kustomization.yaml
        ├── patch-cluster.yaml   # 1 인스턴스, local-path, NodePort 30432
        └── secrets.yaml         # app/superuser 자격증명
```

## 사전 준비

```bash
# CloudNativePG Operator 설치 (클러스터당 1회)
# → helm/cloudnative-pg/README.md 참고
```

## 배포

```bash
kubectl kustomize kustomize/postgres/overlays/sandbox | ssh sandbox001 "kubectl apply -f -"
```

## 접속

```bash
# 내부 (클러스터 내)
psql -h postgres-rw.sandbox.svc.cluster.local -p 5432 -U sandbox -d sandbox

# 외부 (SSH 터널 경유)
# ProxyCommand 설정 후:
psql -h sandbox001 -p 30432 -U sandbox -d sandbox
```

## 관리 DB

| DB | 용도 | 생성 명령 |
|----|------|-----------|
| sandbox | 기본 DB | 자동 생성 (bootstrap) |
| juicefs | JuiceFS 메타데이터 | `CREATE DATABASE juicefs OWNER sandbox;` |
| airflow | Airflow 메타데이터 | `CREATE DATABASE airflow OWNER sandbox;` |

## 참고

- 인스턴스: 1개 (sandbox 환경)
- 스토리지: `local-path` StorageClass, 10Gi
- 외부 접근: NodePort 30432 + SSH 터널
- 자격증명: `secrets.yaml` 참조
