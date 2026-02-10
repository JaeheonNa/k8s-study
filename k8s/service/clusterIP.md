##### Pod는 죽었다 살아나거나 할 때, IP가 계속 바뀜.
##### ClusterIP는 IP가 고정돼 있으며, Pod가 교체될 때마다 알아서 Pod에 연결함.
##### 따라서 ClusterIP를 설정해놓고, ClusterIP의 IP만 알고 있으면, Pod의 IP가 무엇이든 상관 없이 Pod에 요청을 할 수 있음.
##### 단, ClusterIP는 Pod의 IP와 같이 외부에서는 접근이 안 되고, 클러스터 내부에서만 접근이 됨.

    ||||||||||||||||||||||||||  
    |              -> PodA   |  
    |   ClusterIP            |  
    |              -> PodB   |  
    ||||||||||||||||||||||||||  

    apiVersion: v1
    kind: Service
    metadata:
        name: svc-1
    spec:
        selector:
            app: pod
        ports:
        - port: 9000
          targetPort: 8080
        type: ClusterIP

##### 클러스터 내부에서만 접근할 수 있따는 특성상 인가된 사용자(관리자), 내부 대시보드만 ClusterIP에 접근함.
##### Pod의 서비스 상태 디버깅을 할 때 사용됨.