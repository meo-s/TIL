# Kubernetes/Taint And Toleration

Taint는 Node에 적용되는 속성으로, Node에 스케줄링될 Pod를 제한할 수 있는 기능을 제공한다. Toleration은 Pod에 적용되는 속성이다. Taint와 Toleration을 사용하면 특정 Node에 원하는 Pod을 우선적으로 스케줄링하거나 해당 Pod만을 스케줄링하도록 설정할 수 있다.

## Taint 설정 방법

```
$ kubectl taint nodes NODE_NAME KEY=[VALUE]:EFFECT    # NODE_NAME Node에 Taint를 추가한다.
$ kubectl taint ndoes NODE_NAME KEY=[VALUE]:EFFECT-   # NODE_NAME Node에서 Taint를 제거한다. 
```

``` yaml
apiVersion: v1
kind: Node
metadata:
  name: some-node
  ...
spec:
  taints:
  # kubectl taint nodes some-node node-role.kubernetes.io/master=:NoSchedule
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
  ...
status:
  ...
```

`EFFECT`는 `PreferNoSchedule`, `NoSchedule`, `NoExecute` 값 중 하나로 설정할 수 있다.
* `PreferNoSchedule`: Taint에 대응되는 Toleration 속성을 가진 Pod을 우선적으로 스케줄링한다. 상황에 따라 Toleration이 없는 Pod이 스케줄링될 수도 있다.
* `NoSchedule`: Taint에 대응되는 Toleration 속성을 가진 Pod*만*을 스케줄링한다.
* `NoExecute`: `NoSchedule` 효과에 더해 Taint가 설정되기 이전에 이미 스케줄링된 Pod이 있는 경우, 해당 Pod이 새로 설정된 Taint에 대응되는 Toleration 속성을 지니지 않았다면 Pod를 제거한다.

## Toleration 설정 방법

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: some-pod
spec:
  tolerations:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
  ...
status:
  ...
```

## References

[Taints and Tolerations, kubernetes.io](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
