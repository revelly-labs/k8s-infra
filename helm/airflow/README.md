# Apache Airflow 3.x

Airflow 3.x를 Helm으로 배포합니다 (KubernetesExecutor).

## 구조

```
helm/airflow/
└── phase/
    └── sandbox/
        ├── values.yaml          # Executor, 리소스, DAG/로그 설정, Ingress
        └── values-secret.yaml   # DB 연결, 관리자 계정
```

## 사전 준비

```bash
# PostgreSQL에 airflow DB 생성
ssh sandbox001 "kubectl exec -n sandbox postgres-1 -- \
  psql -U postgres -c \"CREATE DATABASE airflow OWNER sandbox;\""

# DAGs용 JuiceFS PVC 생성
ssh sandbox001 "kubectl apply -f -" <<'EOF'
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: airflow-dags
  namespace: sandbox
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: juicefs-retain
  resources:
    requests:
      storage: 1Gi
EOF
```

## 설치

```bash
helm repo add apache-airflow https://airflow.apache.org
helm repo update
ssh sandbox001 "KUBECONFIG=/etc/rancher/k3s/k3s.yaml helm install airflow apache-airflow/airflow \
  -n sandbox -f - -f -" \
  < <(cat helm/airflow/phase/sandbox/values.yaml) \
  < <(cat helm/airflow/phase/sandbox/values-secret.yaml)
```

실제로는 두 파일을 합쳐서 전달:

```bash
cat helm/airflow/phase/sandbox/values.yaml helm/airflow/phase/sandbox/values-secret.yaml | \
  ssh sandbox001 "KUBECONFIG=/etc/rancher/k3s/k3s.yaml helm install airflow apache-airflow/airflow \
  -n sandbox -f -"
```

## 접속

- Web UI: https://airflow.revelly.io
- 자격증명: `values-secret.yaml` 참조

## 주요 설정

| 항목 | 값 |
|------|-----|
| Executor | KubernetesExecutor |
| DB | PostgreSQL (외부, `postgres-rw.sandbox.svc.cluster.local`) |
| DAGs | JuiceFS PVC (`airflow-dags`, juicefs-retain) |
| Logs | JuiceFS PVC (juicefs-retain, 5Gi) |
| Redis | 비활성화 (KubernetesExecutor 사용) |
| 내장 PG | 비활성화 (외부 PG 사용) |

## DAG 배포

DAGs PVC의 JuiceFS 경로: `/sandbox/airflow-dags`

```bash
# 직접 복사
kubectl cp my_dag.py sandbox/<scheduler-pod>:/opt/airflow/dags/

# GitHub Actions 등으로 JuiceFS S3 Gateway 경유 업로드 가능
aws --endpoint-url https://juicefs.revelly.io \
  s3 cp my_dag.py s3://sandbox/airflow-dags/
```

## 참고

- Airflow 3.x부터 Web UI 컴포넌트가 `webserver` → `apiServer`로 변경
- `defaultUser` 설정은 `webserver.defaultUser` 경로에 위치해야 함
- Example DAGs 비활성화: `AIRFLOW__CORE__LOAD_EXAMPLES=False`
