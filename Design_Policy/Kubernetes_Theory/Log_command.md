# Các lệnh theo dõi log

watch -n 2 'sudo crictl ps | grep etcd'
s
# Xem logs etcd
kubectl logs -n kube-system etcd-node-name

# Ví dụ:
kubectl logs -n kube-system etcd-master-node

# Follow logs (real-time)
kubectl logs -n kube-system etcd-master-node -f

# Xem logs với timestamp
kubectl logs -n kube-system etcd-master-node --timestamps

# Xem logs trong 1 giờ qua
kubectl logs -n kube-system etcd-master-node --since=1h

# Xem 100 dòng cuối
kubectl logs -n kube-system etcd-master-node --tail=100

# List tất cả etcd pods
kubectl get pods -n kube-system | grep etcd

# Hoặc
kubectl get pods -n kube-system -l component=etcd

# SSH vào master node, sau đó:

# Xem logs etcd service
sudo journalctl -u etcd -f

# Xem logs với filter
sudo journalctl -u etcd --since "1 hour ago"

# Xem logs trong khoảng thời gian
sudo journalctl -u etcd --since "2025-01-22 10:00:00" --until "2025-01-22 11:00:00"

# Xem logs với priority (error only)
sudo journalctl -u etcd -p err

# Xem logs reverse (mới nhất trước)
sudo journalctl -u etcd -r

# Export logs ra file
sudo journalctl -u etcd > etcd-logs.txt

# Thường ở /var/log/etcd/
sudo tail -f /var/log/etcd/etcd.log

# Hoặc
sudo less /var/log/etcd/etcd.log

# Grep specific errors
sudo grep -i "error" /var/log/etcd/etcd.log

# Theo dõi real-time với grep
sudo tail -f /var/log/etcd/etcd.log | grep -i "error\|warning"

# Check etcd health
kubectl exec -n kube-system etcd-master-node -- etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint health

# Check etcd status
kubectl exec -n kube-system etcd-master-node -- etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint status --write-out=table

# Xem metrics
kubectl exec -n kube-system etcd-master-node -- etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint status -w json

# Describe pod để xem events và status
kubectl describe pod -n kube-system etcd-master-node

# Xem events của namespace
kubectl get events -n kube-system --sort-by='.lastTimestamp' | grep etcd

#!/bin/bash
# watch-etcd-logs.sh

NAMESPACE="kube-system"
POD_NAME=$(kubectl get pods -n $NAMESPACE -l component=etcd -o jsonpath='{.items[0].metadata.name}')

if [ -z "$POD_NAME" ]; then
    echo "etcd pod not found!"
    exit 1
fi

echo "Watching logs for: $POD_NAME"
echo "================================"

kubectl logs -n $NAMESPACE $POD_NAME -f --tail=50 | while read line; do
    # Highlight errors and warnings
    if echo "$line" | grep -qi "error"; then
        echo -e "\033[0;31m$line\033[0m"  # Red
    elif echo "$line" | grep -qi "warning"; then
        echo -e "\033[0;33m$line\033[0m"  # Yellow
    else
        echo "$line"
    fi
done

# Nếu có Prometheus, query metrics:

# etcd leader changes
rate(etcd_server_leader_changes_seen_total[5m])

# etcd request latency
histogram_quantile(0.99, rate(etcd_disk_wal_fsync_duration_seconds_bucket[5m]))

# etcd database size
etcd_mvcc_db_total_size_in_bytes

# etcd network latency
histogram_quantile(0.99, rate(etcd_network_peer_round_trip_time_seconds_bucket[5m]))

# Check if etcd is running
kubectl get pods -n kube-system | grep etcd

# Check etcd container status
kubectl get pod -n kube-system etcd-master-node -o jsonpath='{.status.containerStatuses[0].state}'

# Check etcd resource usage
kubectl top pod -n kube-system etcd-master-node

# Check etcd configuration
kubectl get pod -n kube-system etcd-master-node -o yaml

# Check etcd volumes
kubectl describe pod -n kube-system etcd-master-node | grep -A 5 "Volumes:"

#!/bin/bash
# watch-all-etcd.sh

for pod in $(kubectl get pods -n kube-system -l component=etcd -o name); do
    echo "=== Logs from $pod ==="
    kubectl logs -n kube-system $pod --tail=20
    echo ""
done
