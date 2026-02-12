##### controller는 pod가 죽거나 재생성될 때, 자신이 갖고 있는 template으로 pod를 생성함.
##### template으로 만드는데, label이 필요한 이유는, template으로 생성된 pod를 controller와 연결시켜놓기 위함임.
##### 물론 template 없이도 selector만으로 따로 생성된 pod와 연결시킬 수는 있음.
##### 다만, 아래 예시처럼 template을 사용하면, image가 업그레이드 됐을 때, template을 수정하는 것 만으로
##### 기존의 pod를 재생성할 때, 새로운 버전의 image를 사용해 pod를 자동으로 올림.
##### 아래 ReplicationController는 Deprecated되었고, ReplicaSet이 대체함.

    apiVersion: v1
    kind: ReplicationController
    metadata:
        name: replication-1
    spec:
        replicas: 1
        selector:
            type: web
        template:
            metadata:
                name: pod-1
                labels:
                    type: web
            spec:
                containers:
                - name: container
                  image: najh0528/hello_8080:v2

##### replicas를 2 이상으로 설정을 하면, controller는 그 수만큼 pod를 유지해 줌.

    apiVersion: v1
    kind: ReplicationController
    metadata:
        name: replication-1
    spec:
        replicas: 3
        selector:
            type: web
        template:
            metadata:
                name: pod-1
                labels:
                    type: web
            spec:
                containers:
                - name: container
                  image: najh0528/hello_8080:v2

##### ReplicationController는 selector와 정확히 일치하는 label을 갖고 있는 pod만 관리할 수 있는 반면,
##### ReplicaSet은 규칙을 지정한 후 규칙에 부합하는 label을 가진 pod을 관리하게 할 수 있음.

    apiVersion: v1
    kind: ReplicaSet
    metadata:
        name: replication-1
    spec:
        replicas: 3
        selector:
            matchLabels:
                type: web
            matchExpressions:
            - {key: ver, operator: Exists}
        template:
            metadata:
                name: pod-1
                labels:
                    type: web
            spec:
                containers:
                - name: container
                  image: najh0528/hello_8080:v2

##### matchExpressions에는 Exists, DoesNotExist, In, NotIn 옵션이 있음.

    Exists: key를 설정하고, 해당 키를 갖고 있는 pod를 관리. 
    DoesNotExist: key를 설정하고, 해당 키가 없는 pod를 관리.
    In: key와 values를 설정하고, key가 일치하며, values 중 하나라도 일치하는 pod를 관리.
    NotIn: key와 values를 설정하고, key가 일치하며, values 중 하나도 일치하지 않는 pod를 관리.