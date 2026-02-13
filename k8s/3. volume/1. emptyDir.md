##### emptyDir은 Volume의 일종으로 Pod 내부에 존재함.
##### 따라서 Pod가 내려가면 그 안의 내용들이 사라지게 됨.
##### Pod 내의 여러 컨테이너들이 공유함.

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
          volumeMounts:
          - name: empty-dir
            mountPath: /mount1
        - name: container-2
          image: najh0528/hello_8080
          ports:
          - containerPort: 8080
          volumeMounts:
          - name: empty-dir
            mountPath: /mount2
        volumes:
        - name: empty-dir
          emptyDir: {}

##### container-1과 container-2의 mountPath가 다르다고 하더라도,
##### 같은 empty-dir volume을 연결했기 때문에,
##### 두 컨테이너가 공유하는 volume이 됨.
##### 단지 두 컨테이너가 바라보는 '디렉토리명'이 다를 뿐, 물리적으로는 같은 디렉토리임.