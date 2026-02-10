##### namespace의 자원 한계를 설정하는 object.
##### resourceQuota가 부여된 namespace에 pod를 스케쥴링 할 때는, pod가 필요로하는 resource를 명시해야 함.
##### pod가 존재하는 가운데 resourceQuota를 만들면, 이미 사용 중인 자원에 대해서는 제한을 걸지 못함.
##### 따라서 resourceQuota를 생성하기 전에 namespace 내 object를 지우고 resourceQuota부터 차례로 생성하는 게 바람직함.

    apiVersion: v1
    kind: ResourceQuota
    metadata:
        name: resourceQuota-1
        namespace: namespace-1
    spec:
        hard:
            requests.memory: 3Gi
            limits.memory: 6Gi
    
    apiVersion: v1
    kind: Pod
    metadata:
        name: pod-1
    spec:
        containers:
        - name: container-1
          image: najh0528/hello_8080
          resources:
            requests:
                memory: 2Gi
            limits:
                memory: 4Gi

##### pod 수를 제한할 수도 있음.

    apiVersion: v1
    kind: ResourceQuota
    metadata:
        name: resourceQuota-1
        namespace: namespace-1
    spec:
        hard:
            pods: 2