# Monitoring (kube-prometheus-stack)

Prometheus, Grafana, Node Exporter, kube-state-metrics를 포함하는 통합 모니터링 스택.

## 구조

```
helm/monitoring/
└── phase/
    └── sandbox/
        ├── values.yaml          # Grafana, Prometheus, Exporter 설정
        └── values-secret.yaml   # Grafana 관리자 계정
```

## 설치

> **참고**: `values.yaml`과 `values-secret.yaml` 모두 `grafana:` 키를 포함하므로
> `cat`으로 합치면 값이 덮어씌워집니다. 반드시 `-f` 플래그를 두 번 사용합니다.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

scp helm/monitoring/phase/sandbox/values.yaml sandbox001:/tmp/monitoring-values.yaml
scp helm/monitoring/phase/sandbox/values-secret.yaml sandbox001:/tmp/monitoring-values-secret.yaml
ssh sandbox001 "KUBECONFIG=/etc/rancher/k3s/k3s.yaml helm install monitoring \
  prometheus-community/kube-prometheus-stack -n sandbox \
  -f /tmp/monitoring-values.yaml -f /tmp/monitoring-values-secret.yaml \
  && rm /tmp/monitoring-values.yaml /tmp/monitoring-values-secret.yaml"
```

## 업그레이드

```bash
scp helm/monitoring/phase/sandbox/values.yaml sandbox001:/tmp/monitoring-values.yaml
scp helm/monitoring/phase/sandbox/values-secret.yaml sandbox001:/tmp/monitoring-values-secret.yaml
ssh sandbox001 "KUBECONFIG=/etc/rancher/k3s/k3s.yaml helm upgrade monitoring \
  prometheus-community/kube-prometheus-stack -n sandbox \
  -f /tmp/monitoring-values.yaml -f /tmp/monitoring-values-secret.yaml \
  && rm /tmp/monitoring-values.yaml /tmp/monitoring-values-secret.yaml"
```

## 접속

- Grafana: https://grafana.revelly.io
- Prometheus: https://prometheus.revelly.io

## 모니터링 대상

| 대상 | 방식 | 비고 |
|------|------|------|
| sandbox001 노드 | Node Exporter | CPU, Memory, Disk, Network |
| Kubernetes | kube-state-metrics | Pod, Deployment, Node 상태 |
| MinIO | ServiceMonitor | `/minio/v2/metrics/cluster` |
| PostgreSQL | PodMonitor | CloudNativePG 내장 메트릭 (port 9187) |
| Airflow | ServiceMonitor | StatsD Exporter |

### Grafana 대시보드

플랫폼별 폴더로 자동 프로비저닝됩니다 (수동 Import 불필요):

| 폴더 | 대시보드 | gnetId | 설명 |
|------|----------|--------|------|
| Node | Node Exporter Full | 1860 | 서버 리소스 상세 (CPU, Memory, Disk, Network) |
| MinIO | MinIO Overview | 13502 | MinIO 클러스터 메트릭 |
| CloudNativePG | CloudNativePG | 20417 | PostgreSQL 클러스터 |
| Airflow | Airflow Cluster | 20994 | Scheduler, Worker, DAG Processor |
| Airflow | Airflow DAG | 20789 | DAG별 실행 통계 |
| Kubernetes | (기본 대시보드) | — | API Server, Compute Resources, Networking 등 |

> **참고**: k3s 환경에서 메트릭을 노출하지 않는 컴포넌트(etcd, Controller Manager, Scheduler, Proxy)는 비활성화되어 있습니다.

## 삭제

```bash
ssh sandbox001 "KUBECONFIG=/etc/rancher/k3s/k3s.yaml \
  helm uninstall monitoring -n sandbox"

# CRD 수동 삭제 (필요시)
ssh sandbox001 "KUBECONFIG=/etc/rancher/k3s/k3s.yaml \
  kubectl delete crd prometheuses.monitoring.coreos.com \
  prometheusrules.monitoring.coreos.com \
  servicemonitors.monitoring.coreos.com \
  podmonitors.monitoring.coreos.com \
  alertmanagers.monitoring.coreos.com \
  alertmanagerconfigs.monitoring.coreos.com \
  probes.monitoring.coreos.com \
  scrapeconfigs.monitoring.coreos.com \
  thanosrulers.monitoring.coreos.com"
```

## 참고

- 릴리스명: `monitoring`, 네임스페이스: `sandbox`
- Alertmanager는 비활성화 상태 (추후 Slack/Email 알림 연동 시 활성화)
- Prometheus 데이터 보존: 15일
- `serviceMonitorSelectorNilUsesHelmValues: false`로 설정하여 모든 네임스페이스의 ServiceMonitor 수집
