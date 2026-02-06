# Master Node Init

### [4] Ubuntu 기본 설정
### [4-1] 패키지 업데이트
    sudo apt update && sudo apt upgrade -y

### [4-2] 타임존 설정
    sudo timedatectl set-timezone Asia/Seoul

### [4-3] 필수 패키지 설치
    sudo apt update
    sudo apt install -y apt-transport-https ca-certificates curl iproute2 gnupg2

### [4-4] hosts 설정
###### 중복 추가 방지를 위해 확인 후 추가 권장
    sudo tee -a /etc/hosts << EOF
    192.168.2.2 k8s-master
    192.168.2.4 k8s-node1
    192.168.2.5 k8s-node2
    EOF

### [5] kubeadm 설치 전 사전작업
### [5] Swap 비활성화
    sudo swapoff -a && sudo sed -i '/ swap / s/^/#/' /etc/fstab

### [6] 컨테이너 런타임 설치
### [6-1] 컨테이너 런타임 설치 전 사전작업 (커널 모듈 및 sysctl)
##### 수정 포인트: sudo와 modprobe 경로 확인
    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF
    
    sudo modprobe overlay
    sudo modprobe br_netfilter

##### 수정 포인트: ip_forward 설정을 확실하게 적용
    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward                 = 1
    EOF
    
    sudo sysctl --system

### [6-2] 컨테이너 런타임 (containerd 설치)
### [6-2-1] Docker GPG 키 및 저장소 추가
    sudo mkdir -p /usr/share/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    
    echo '======== [6-2-2] containerd 설치 ========'
    sudo apt update
    sudo apt install -y containerd.io
    sudo systemctl enable --now containerd

### [6-3] 컨테이너 런타임 : CRI 설정 및 SystemdCgroup 활성화
##### 수정 포인트: 'sudo > sudo' 오타 수정 및 경로 확실화
    sudo mkdir -p /etc/containerd
    containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
    sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
    sudo systemctl restart containerd

### [7] Kubernetes 설치
### [7-1] Kubernetes GPG 키 및 저장소 추가
##### 수정 포인트: v1.30 최신 저장소 방식 적용
    sudo mkdir -p /usr/share/keyrings
    sudo rm -f /usr/share/keyrings/kubernetes-apt-keyring.gpg
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /usr/share/keyrings/kubernetes-apt-keyring.gpg
    
    echo 'deb [signed-by=/usr/share/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

### [7-2] kubelet, kubeadm, kubectl 패키지 설치
    sudo apt update
    sudo apt install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl
    sudo systemctl enable --now kubelet

### [8] kubeadm으로 클러스터 생성
### [8-1] 클러스터 초기화
##### 수정 포인트: 이미 실행 중인 경우를 대비해 reset 후 진행 권장
    sudo kubeadm reset -f
    sudo kubeadm init \
      --pod-network-cidr=20.96.0.0/16 \
      --apiserver-advertise-address=192.168.2.2 \
      --kubernetes-version=v1.30.0

### [8-2] kubectl 사용 설정
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

##### Join 명령어 저장 (kubeconfig 지정하여 에러 방지)
    sudo kubeadm token create --print-join-command --kubeconfig /etc/kubernetes/admin.conf > ~/join.sh
    chmod +x ~/join.sh

### [8-3] Pod Network 설치 (Calico v3.28+)
##### 1. Calico Operator 설치
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/tigera-operator.yaml

##### 2. Custom Resources 설정 (CIDR 수정 포함)
    curl -L https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/custom-resources.yaml -O
    sed -i 's/192.168.0.0\/16/20.96.0.0\/16/g' custom-resources.yaml
    kubectl create -f custom-resources.yaml


### [9] 쿠버네티스 편의기능 설치
### [9-1] kubectl 자동완성 기능
    apt install -y bash-completion
    echo "source <(kubectl completion bash)" >> ~/.bashrc
    echo 'alias k=kubectl' >>~/.bashrc
    echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
    source ~/.bashrc

### [9-2] Dashboard 설치 (v1.30 호환)
##### 공식 리포지토리의 최신 안정 버전(v3.0.0-alpha 이상 또는 v2.7.0 유지 가능하나 최신 권장)
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
    kubectl patch svc kubernetes-dashboard -n kubernetes-dashboard -p '{"spec": {"type": "NodePort"}}'
    kubectl get svc -n kubernetes-dashboard --> 대시보드 포트 확인
    kubectl create serviceaccount admin-user -n kubernetes-dashboard
    kubectl create clusterrolebinding admin-user-binding \
      --clusterrole=cluster-admin \
      --serviceaccount=kubernetes-dashboard:admin-user
    kubectl -n kubernetes-dashboard create token admin-user


### [9-3] Metrics Server 설치 (v0.7.1)
##### 공식 릴리스 사용
    kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
##### Metrics Server가 사설 인증서를 무시하도록 설정 추가
    kubectl patch deployment metrics-server -n kube-system --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'

### [10-1] Pod 상태 확인
    kubectl get pod -A