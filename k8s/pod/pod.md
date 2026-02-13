##### 하나의 Pod 안에는 여러 컨테이너가 존재할 수 있음.
##### 단, 하나의 Pod 안에 존재하는 컨테이너는 port를 하나만 가질 수 있으며, 다른 컨테이너와 중복될 수 없음.

    apiVersion: v1
    kind: Pod
    metadata:
        name: pod-1
        labels:
            app: pod
    spec:
        containers:
        - name: container-1
          image: najh0528/hello_8000
          ports:
          - containerPort: 8000
        - name: container-2
          image: najh0528/hello_8080
          ports:
          - containerPort: 8080
        terminationGracePeriodSeconds: 0

##### pod를 삭제하면 기본적으로 30초 후에 삭제가 됨.
##### terminationGracePeriodSeconds 옵션을 사용하면, 삭제 시 몇 초 후에 삭제할지 직접 설정할 수 있음. 