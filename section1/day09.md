# Day09

# Secret

```yaml
kubectl get secret
```

```yaml
kubectl describe secrets dashboard-token
```

```yaml
kubectl get svc
kubectl get secrets
```

```yaml
kubectl create secrets generic db-secret \
						--from-literal=DB_Host=sql-1 \
						--from-literal=DB_User=root  \
						--from-literal=DB_Password=password123
```

```yaml
kubectl edit pod webapp-pod

envFrom:
	- secretRef:
		name: db-secret
```

---

# Encrypting Secret Data at Rest

- secret 오브젝트 생성한다.
- kubectl create secret으로 일반적인 secret을 생성한다.

```yaml
kubectl create secret generic my-secret --from-literal=key=supersecret

kubectl get secret
kubectl describe secret my-secret
kubectl get secret my-secret -o yaml
```

- 인코딩된 형식으로 볼 수 있다. 이것은 누구나 해독 가능하다.

```yaml
echo "cjdjejsljfjowf" | base64 --decode
```

- secret 정의 파일 대신 Github 또는 다른 것으로 푸쉬하는게 좋다.
    - 누구나 파일을 선택하고 커맨드를 실행하면 secret을 볼 수 있기 때문이다.
- secret 데이터를 etcd 서버에 저장되는 것을 확인해보자.

```yaml
etcdctl

# etcdctl 설치
apt-get install etcd-client

# etcd 서버가 파드에서 실행 중이다.
# Pod 안에서 etcdctl 커맨드를 실행할 수 있다.
kubectl get pods -n kube-system
etcd-controlplane

# 커맨드 실행
ls /etc/kubernetes/etcd/
```

- ETCD에 저장된 현재 가지고 있는 secret, 암호 확인하기

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/59884de8-873b-4d3b-b839-ecc02cace348/Untitled.png)

# Scale Application

- 대규모 모놀리식 애플리케이션을 분리한 MSA에 대해 알아보자.
- 이 아키텍쳐는 전체 애플리케이션이 아닌 각 서비스를 확장, 축소, 수정한다.

- 웹 서버 인스턴스 당 하나의 에이전트 인스턴스가 필요하다.
- 다중 컨테이너 Pod에서 이를 수행할 수 있는데, 함께 확장 및 축소할 수 있어야 한다.
- 다중 컨테이너 파드는 라이프 사이클을 함께한다. 같이 만들어지고, 같이 죽는다.
    - 같은 네트워크 공간을 공유하고, 서로를 로컬 호스트로 참조할 수 있다. 또한 동일한 볼륨에 액세스할 수 있다.

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp
	labels: 
		name: simple-webapp
spec:
	containers:
	- name: simple-webapp
		image: simple-webapp
		ports:
			- containerPort: 8080
	- name: log-agent
		image: log-agent
```

---

# InitContainers

- 다중 컨테이너 파드에서 각 컨테이너는 파드의 수명주기 동안 살아있는 프로스세를 실행할 것으로 예상된다.
- 하지만, 컨테이너를 완료될 때까지 실행되는 프로세스를 실행하고 싶을 수 있다.
- 이것을 initContainers로 할 수 있다.

- 파드가 처음 생성되면 initContainer가 실행되고, initContainer의 프로세스는 애플리케이션을 호스팅하는 실제 컨테이너가 시작되기 전에 완료되어야 한다.
- initContainer는 순차적으로 한 번에 하나씩 실행된다.
- initContainer 중 하나라도 완료되지 않으면 쿠버네티스는 initContainer가 성공할 때까지 파드를 반복해서 다시 시작한다.
