### multipass 설치
    brew install multipass

### VM 생성
    multipass launch --name master --cpus 2 --memory 2G --disk 20G
    multipass launch --name worker01 --cpus 2 --memory 2G --disk 20G
    multipass launch --name worker02 --cpus 2 --memory 2G --disk 20G

### VM 확인
    multipass list

### VM 접속
    multipass shell <VM명>
