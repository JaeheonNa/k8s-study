##### Pod를 '특정 Node'에 위치 시킬 수 있음.
##### nodeSelector의 값은 '특정 Node'의 label임.
##### node의 label은 kubectl describe node <Node명> 명령어를 통해 확인 가능.

    apiVersion: v1
    kind: Pod
    metadata:
        name: pod-1
        labels:
            app: pod
    spec:
        nodeSelector:
            kubernetes.io/hostname: node01
        containers:
        - name: container-1
          image: najh0528/hello_8000
          ports:
          - containerPort: 8000