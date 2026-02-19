# Worker Node Init

### [4] Ubuntu 기본 설정
### [4-2] 타임존 설정
    sudo timedatectl set-timezone Asia/Seoul

### [4-3] 필수 패키지 설치
    sudo apt update
    sudo apt install -y apt-transport-https ca-certificates curl gnupg2

### [4-4] hosts 설정
##### 마스터에서 설정한 IP와 동일하게 맞춰주세요.
    sudo tee -a /etc/hosts << EOF
    192.168.2.2 k8s-master
    192.168.2.4 k8s-node1
    192.168.2.5 k8s-node2
    EOF

### [5] kubeadm 설치 전 사전작업
### [5-1] Swap 비활성화 (필수)
    sudo swapoff -a && sudo sed -i '/ swap / s/^/#/' /etc/fstab

### [6] 컨테이너 런타임 설치
### [6-1] 커널 모듈 및 sysctl 세팅
    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF
    
    sudo modprobe overlay
    sudo modprobe br_netfilter
    
    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward                 = 1
    EOF
    
    sudo sysctl --system

### [6-2] containerd 설치 (Docker Repo 활용)
    sudo mkdir -p /usr/share/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    
    sudo apt update
    sudo apt install -y containerd.io
    sudo systemctl enable --now containerd

### [6-3] 컨테이너 런타임 : CRI 설정 및 SystemdCgroup 활성화
    sudo mkdir -p /etc/containerd
    containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
    sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
    sudo systemctl restart containerd

### [7] Kubernetes v1.30 설치
### [7-1] GPG 키 및 저장소 추가
    sudo rm -f /usr/share/keyrings/kubernetes-apt-keyring.gpg
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /usr/share/keyrings/kubernetes-apt-keyring.gpg
    
    echo 'deb [signed-by=/usr/share/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

### [7-2] 패키지 설치 
    sudo apt update
    sudo apt install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl
    sudo systemctl enable --now kubelet

### [8] 클러스터 조인 (Join) 
##### 마스터 노드에서 생성했던 ~/join.sh 파일의 내용을 여기에 실행.
##### ex)
    마스터 노드: cat ~/join.sh + (ctrl+c)
    워커 노드: (ctrl+v) + enter