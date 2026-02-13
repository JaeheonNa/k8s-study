##### 한 namepace에서는 같은 이름을 갖는 pod를 생성할 수 없음.
##### namespace는 각자 필요한 resource를 할당받아 사용함.
##### 서로 다른 namespace에 존재하는 pod과 service는 연결될 수 없음.
##### pv나 node는 namespace끼리 공유할 수 있는 자원임.

    apiVersion: v1
    kind: Namespace
    metadata:
        name: namespace-1