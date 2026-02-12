
# Lưu ý khi Upgrade Client Version, Server Version, kubeadm version

## Quy trình cài đặt

Nhớ khi cài thì nếu dùng kubeadm, kubelet, kubectl thì phải cài cùng 1 phiên bản ngay từ đầu trước. 


## Quy trình upgrade 

- Luôn upgrade Master node trước

Quy trình upgrade kubenetes version 

B1: Upgrade master node - control plane 
 
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key \
| sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

```
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /" \
| sudo tee /etc/apt/sources.list.d/kubernetes-v1.31.list
```
```
sudo apt update
```
```
sudo apt-mark unhold kubeadm
sudo apt install -y kubeadm=1.31.* --allow-downgrades 
sudo apt-mark hold kubeadm
```
```
kubeadm version
```
```
sudo kubeadm upgrade plan
```
```
# ( chờ master node ready mới dùng lệnh này tiếp ) 
sudo kubeadm upgrade apply v1.31.y
```
```
sudo apt-mark unhold kubelet kubectl
sudo apt install -y kubelet=1.31.* kubectl=1.31.* --allow-downgrades
sudo apt-mark hold kubelet kubectl
sudo systemctl restart kubelet
```

**Nếu NotReady → kiểm tra kubelet + CNI**

## Phiên bản của Client Version không trùng với Server version

1. Upgrade Server Version 

## Một số lỗi khác

### Nếu `coredns` mà mãi không running thì khả năng CNI gặp vấn đề 

Cài lại coredns
- ```kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml``` 
- ```kubectl get pods -n kube-system | grep calico```
- Chờ tới khi running 

# Lưu ý trong quá trình vận hành kubernetes

## Về version

- Các phiên bản của Client Version và Server Version phải trùng với nhau

## Về Upgrade

- Phải Upgrade Master node trước 

## Về việc dùng K8s Cluster thì phải cấu hình ip_forward trên VM là Control-Plane

```
itachi0737@instance-20260110-064917:~/deploy-k8s-ierp$ cat /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
```

## Về việc đầy Disk của Virtual Machine 

- Dấu hiệu nhận biết: connection refused khi dùng lệnh `kubectl`, unknow server, permission denied admin.conf
- Root Cause: Disk bị đầy -> hệ thống bị gián đoạn, VM bị full disk, etcd/kube-apiserver restart nhiều lần. Nhưng sau khi resize disk, restart service thì trạng thái Cert + kubeconfig bị lệch
- Giải pháp:
```
# Regenerate lại kubeconfig admin ( không reset cluster )
sudo kubeadm init phase kubeconfig admin
# Sau đó copy lại 
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
chmod 600 ~/.kube/config
```
```
kubectl
  |
  | 1. kubeconfig (server / cert / user)
  v
TCP connection (6443)
  |
  | 2. Network / Firewall / IP bind
  v
kube-apiserver process
  |
  | 3. TLS / cert / auth
  v
etcd
  |
  | 4. Storage / Disk / IO
  v
Kubernetes control plane OK
```

## Xử lý lỗi khi Cluster ở trạng thái `Control plane crash loop`

1. Kiểm tra xem có đủ 4 file này không
```
tu@tu-Virtual-Machine:~$ sudo ls -l /etc/kubernetes/manifests
total 16
-rw------- 1 root root 2622 ມ.ກ. 22 16:38 etcd.yaml
-rw------- 1 root root 3959 ມ.ກ. 22 16:38 kube-apiserver.yaml
-rw------- 1 root root 3344 ມ.ກ. 22 16:38 kube-controller-manager.yaml
-rw------- 1 root root 1726 ມ.ກ. 22 16:38 kube-scheduler.yaml
```
2. Xóa log nếu đầy disk, nhưng với yêu cầu trên Production là log phải được streaming để đưa ra Centralize Logging rồi.