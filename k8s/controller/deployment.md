##### 기본적인 배포 전략들
    recreate: 기존 pod를 전부 죽인 후 새 pod 생성. Downtime 존재.
    rolling update: 모든 pod가 신규 버전의 pod가 될 때까지 하나씩 순차적으로 새 pod를 생성한 뒤, 기존 pod 삭제하는 절차를 반복.
    blue green: 새 pod를 생성한 뒤, 기존 pod에 연결돼 있던 네트워크를 모두 새 pod로 연결. 새 pod에 문제가 생기면, 기존 pod로 네트워크 연결 복원. 새 pod에 문제가 없다고 판단됐을 때, 기존 pod 삭제. 
    canary: 새 pod를 하나 생성해서, 무작위로 일부 inbound가 새 pod로 연결되도록한 후, 문제가 없는 것으로 판단되면, 기존 pod들을 삭제 후 새 pod 배포.

##### Deployment는 replicaSet을 control하는 방식을 통해 위 배포 전략들을 지원하기 위한 object임. 
##### recreate 방식으로 Deployment를 생성하면, Deployment에서 pod의 버전을 변경 시, 기존 ReplicaSet의 replicas는 0이 되고, 새 ReplicaSet이 생성되면서 pod가 생성됨.
##### revisionHistoryLimit 옵션은 replicas가 0인 ReplicaSet을 몇 개 남겨둘지에 대한 옵션임.
    
    apiVersion: apps/v1
    kind: Deployment
    metadata:
        name: deployment-1
    spec:
        selector:
            matchLabels:
                type: app
        replicas: 2
        strategy:
            type: Recreate
        revisionHistoryLimit: 1
        template:
            metadata:
                type: app
            spec:
                containers:
                - name: container
                  image: najh0528/hello_8000
    
    apiVersion: v1
    kind: Service
    metadata:
        name: svc-1
    spec:
        selector:
            type: app
        ports:
            - port: 8080
              protocol: TCP
              targetPort: 8080

##### rolling update 방식으로 Deployment를 생성하면, Deployment에서 pod의 버전을 변경 시, 기존 ReplicaSet의 replicas가 1 줄어들고, 새 ReplicaSet이 생성되면서 pod가 생성됨.
##### 새 pod가 running 상태가 되면, 기존 ReplicaSet의 replicas가 또 다시 1 줄어들고, 새 ReplicaSet의 replicas가 1 증가함. 기존 ReplicaSet의 replicas가 0이 될 때까지 반복됨.
##### 이 과정에서 새 ReplicaSet이 기존 pod와 연결될 수 있기 때문에 Deployment는 새 ReplicaSet을 생성할 때, matchLabel를 추가함.
##### 참고로 rolling update가 default 임.


    apiVersion: apps/v1
    kind: Deployment
    metadata:
        name: deployment-1
    spec:
        selector:
            matchLabels:
                type: app
        replicas: 2
        strategy:
            type: RollingUpdate
        minReadySeconds: 10
        template:
            metadata:
                type: app
            spec:
                containers:
                - name: container
                  image: najh0528/hello_8000
    
    apiVersion: v1
    kind: Service
    metadata:
        name: svc-1
    spec:
        selector:
            type: app
        ports:
            - port: 8080
              protocol: TCP
              targetPort: 8080
