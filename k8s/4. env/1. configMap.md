###### configMap은 key-value로 만들어져 있음.
###### 필요한 상수를 정의해두면, pod를 띄울 때, container의 환경변수로 세팅을 할 수 있음.

    apiVersion: v1
    kind: ConfigMap
    metadata:
        name: cm-dev
    data:
        key : 'value'

    apiVersion: v1
    kind: Pod
    metadata:
        name: pod-1
    spec:
        containers:
        - name: container
          image: najh0528/hello_8000
          envFrom:
          - configMapRef:
                name: cm-dev

###### 파일 기반으로 configMap을 만들고, 이를 직접 주입할 수도 있음.

    kubectl create configMap <configMap명> --from-file=<파일>

    apiVersion: v1
    kind: Pod
    metadata:
        name: pod-1
    spec:
        containers:
        - name: container
          image: najh0528/hello_8000
          env:
          - name: file
            valueFrom:
                configMapKeyRef:
                    name: <configMap명>
                    key: <파일명>

###### 파일을 직접 마운트 시킬 수도 있음.
###### 파일을 직접 마운트 시키는 경우, 파일의 내용이 바뀌면, 변경된 내용이 반영됨. (위의 경우는 변경되지 않음))

    apiVersion: v1
    kind: Pod
    metadata:
        name: mount
    spec:
        containers:
        - name: container
          image: najh0528/hello_8000
          volumeMounts:
          - name: file-volume
            mountPath: /mount
        volumes:
        - name: file-volume
          configMap:
            name: <configMap명>