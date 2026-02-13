##### Pod에 label을 '키-값'쌍으로 부여하면, k8s 구성 요소들이 해당 pod를 label로 식별할 수 있음.

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