##### resourceQuota가 namespace의 자원 한계를 설정하는 거라면,
##### limitRange는 namespace에 스케쥴링 될 수 있는 pod의 한계를 설정하는 object.

    apiVersion: v1
    kind: LimitRange
    metadata:
        name: limitRange-1
        namespace: namespace-1
    spec:
        limits:
        - type: Container
          min:
            memory: 1Gi
          max:
            memory: 4Gi
          maxLimitRequestRatio:
            memory: 3
          defaultRequest:
            memory: 1Gi
          default:
            memory: 2Gi


##### 위 LimitRange의 예시에 따르면
##### namespace-1에 스케쥴링 될 pod는
##### 최소 1Gb 메모리를 요구해야하며, 최대 4Gb 메모리 사용량을 가질 수 있으나, min-max의 크기가 3배 이상이 될 수는 없음.
##### 만약 requests.memory와 requests.limit 설정 없이 스케쥴링 될 경우에는 각각 1Gi, 2Gi로 설정되어 스케쥴링 됨.