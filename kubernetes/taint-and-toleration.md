# Kubernetes/Taint And Toleration

Taint는 Node에 적용되는 속성으로, Node에 스케줄링될 Pod를 제한할 수 있는 기능을 제공한다.  반대로 Toleration은 Pod에 적용되는 속성이다. Taint와 Toleration을 사용하면 특정 Node에 원하는 종류의 Pod을 우선적으로 스케줄링하거나 해당 Pod만을 스케줄링하도록 유도할 수 있다.

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

위의 설정에서 `some-pod`은 `node01` Node의 `node-role.kubernetes.io/master=:NoSchedule` Taint에 대응되는 Toleration을 지고 있다. 따라서 `some-pod`은 `node01`에 스케줄링될 수 있다.

`kubectl` CLI를 사용하여 현재 Pod들의 상태를 확인해보자.

```
$ kubectl get po -o wide
NAME     READY   STATUS    RESTARTS   AGE    IP             NODE 
some-pod 1/1     Running   0          6m9s   172.16.39.82   node01
```

Toleration을 설정할 때 `tolerationSeconds` 속성을 지정할 수도 있다. `tolerationSeconds`는 내성(Toleration)의 지속 시간을 의미한다. 어떤 Pod의 `tolerationSeconds`를 60으로 설정했다고 가정하자. 이 Pod이 해당 Toleration에 대응되는 Taint를 가진 Node에 스케줄링되었다면, 해당 Pod은 60초 이후에 Node로부터 제거된다. 내성의 지속 시간인 `tolerationSeconds`가 60초이기 때문이다.

``` yaml
apiVersion: v1
kind: Pod
metadata:
  ...
spec:
  tolerations:
  # tolerationSeconds가 60이므로 evolving-taint를 가지는 Node에 할당되면 60초 이후 Node로부터 제거된다.
  - effect: NoExecute
    key: evolving-taint
    tolerationSeconds: 60

  # effect를 지정하지 않으면 모든 종류의 effect에 적용된다.
  - key: some-taint-00
    value: value-of-some-taint-00

  # operator를 Exists로 설정함으로써 Taint의 value에 무관하게 Toleration이 적용되도록 설정할 수 있다.
  - key: some-taint-01
    operator: Exists
status:
  ...
```

## References

[Taints and Tolerations, kubernetes.io](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
