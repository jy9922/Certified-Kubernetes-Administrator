# Replication Controller

- 컨트롤러는 쿠버네티스의 두뇌이다.
- 쿠버네티스 오브젝트를 모니터링하고 반응하는 프로세스이다.

## Replica

- 애플리케이션을 실행하는 단일 파드가 있다.
- 어떤 이유로 애플리케이션 파드가 죽었다고 가정하자.
- 유저는 더이상 애플리케이션에 접근할 수 없다.
- 위와 같은 상황을 방지하고자 우리는 동시에 실행되는 둘 이상의 인스턴스 또는 파드를 가져야 한다.
- 그러면 하나가 실패해도 여전히 다른 애플리케이션에서 실행중인 애플리케이션이 있기 때문이다.

![image](https://user-images.githubusercontent.com/76610641/219092817-a138a37e-021b-4654-8dbf-6d5a9a865b5c.png)

### Replication Controller가 필요한 이유

### HA

- Replication Controller은 클러스터에서 파드의 여러 인스턴스를 실행하는데 도움이 된다.
- 이는 곧 **고가용성**을 제공한다.

❓ ***만약 파드 한 개만 사용할 예정이라면 Replication Controller를 사용할 수 없는 것일까?***

아니다. 파드가 하나만 있어도 Replication Controller이 도움이 될 수 있다.

- Replication Controller는 **기존 파드가 죽었을 때, 새로운 파드를 자동으로 불러와서 지정된 수의 파드가 항상 실행 중이라는 것을 보장**한다.

### Load Balancing & Scaling

![image](https://user-images.githubusercontent.com/76610641/219092916-2015bf71-1a01-4d97-83ce-14617ef295f4.png)

- Replication Controller가 필요한 또다른 이유는 **부하를 분산하기 위해 여러 파드를 생성**해야하기 때문이다.
- 예를 들어, 유저들에게 서비스를 제공하는 하나의 파드가 있다.
    - 유저가 늘어나면 로드밸런싱을 위해 추가 파드를 배포한다.
    - 해당 노드의 리소스가 부족해지면 클러스터 내 다른 노드에 추가 파드를 배포할 수 있다.
- Replication Controller는 클러스터의 여러 노드에 걸쳐있기 때문에 여러 노드에서 파드와 애플리케이션의 로드 밸런싱하는데 도움이 된다.

## ReplicaSet과 Replication Controller 차이

- 둘 다 같은 목적을 갖고 있지만 같은 것은 아니다.
- Replication Controller는 더 오래된 기술이며, ReplicaSet에 의해 대체되었다.
- ReplicaSet은 Replication을 설정하는 새로운 권장 방법이다.
- HA, Load Balancing, Scailing은 두 기술 모두 적용 가능하다. 그리고 각각의 작동 방식에는 약간의 차이가 있다.

### Creating Replication Controller

- YAML 파일 생성
    - 4개의 루트 레벨 속성
        - apiVersion: API 버전은 우리가 만드는 항목에 따라 다르다. Replication Controller는 Kubernetes API v1에서 지원된다.
        - kind: ReplicationController이다.
        - metadata: 딕셔너리이다. name과 labels를 지정한다.
        - spec: Pod 복제를 위한 Pod template을 정의해야한다. 그리고 복제본 수에 해당하는 replicas가 있다.
        
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
	name: myapp-rc
	labels:
		app: myapp
		type: front-end
spec:
	replicas: 3
	template:
		metadata:
			name: myapp-pod
			labels:
				app: myapp
				type: frontend
		spec:
			containers:
			- name: nginx-container
				image: nginx
```

```bash
kubectl create -f rc-definition.yaml

# 확인
kubectl get replicationcontroller

# Pod 확인
kubectl get pods
# replication controller 이름으로 시작되는 Pod 3개 확인
```

### Creating ReplicaSet

- YAML 파일 생성
    - apiVersion: **apps/v1**
    - kind: ReplicaSet
    - metadata:
        - name:
        - labels:
    - spec:
        - template:
        - replicas:
        - **selector: 어떤 파드가 ReplicaSet 아래에 있는지 식별하는 역할**
    
    <aside>
    ⚙️ Replication Controller와 ReplicaSet의 큰 차이점
    바로 selector의 유무이다. spec 섹션의 template에 포함시켰는데 왜 selector 섹션을 작성해야 할까? 
    ReplicaSet이 ReplicaSet을 생성할 때 함께 만들어진 파드가 아닌 파드도 관리가 가능하기 때문이다. 
    예를 들어, ReplicaSet 생성 이전에 있었던 파드들이 selector와 labels와 매칭시켜 ReplicaSet이 Replica를 만들 때 해당 파드를 고려하여 Replica를 만들게 된다.
    
    </aside>
    
- selector은 ReplicationController와 ReplicaSet 사이의 큰 차이점 중 하나다.
- selector은 ReplicationController에서 필수 필드는 아니지만 사용 가능하다.
    - ReplicationController에서 selector를 사용하지 않으면 pod definition file에서 제공된 라벨과 동일하다고 가정한다.
    - ReplicaSet에서 selector은 필수이며, matchLabels를 작성해야한다.
        - matchLabels는 파드의 labels와 매칭되는 정보이다.
        - ReplicaSet의 selector는 또한 matchLabels에서 사용할 수 있는 다른 많은 옵션도 제공한다.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
	    name: myapp-pod
			labels:
				app: myapp
				type: front-end
		spec:
			containers:
			- name: nginx-controller
				image: nginx
	replicas: 3
	selector:
		matchLabels:
			type: front-end
```

```yaml
kubectl apply -f replicaset-definition.yaml
kubectl get replicaset
kubectl get pods
```

## Labels와 Selectors

- Kubernetes에서 파드와 오브젝트에 labels를 지정하는 이유
- 가정
    - 프론트엔드 웹 애플리케이션의 인스턴스를 3개 배포했다고 가정하자.
    - Replication Controller 혹은 ReplicaSet을 생성해서 항상 3개의 Pod가 실행중인 상태로 만들고 싶어한다.
    - 따라서 ReplicaSet은 항상 기존에 이미 존재하고 있는 파드를 모니터링한다.
    - 만약 파드가 없다면 ReplicaSet은 자동으로 파드를 생성해준다.
    - ReplicaSet의 역할은 파드를 모니터링하고, 그들 중 하나라도 실패하면 새로운 파드를 배포하는 것이다.
    - **ReplicaSet은 사실 파드를 모니터링하는 프로세스**이다.
    
    ❓**이때 ReplicaSet은 어떤 파드를 모니터링할지 어떻게 알 수 있을까?**
    
    - 클러스터에 각자의 애플리케이션을 실행하고 있는 수백개의 Pod가 존재한다. 바로 여기서 label이 필요하다.
    - labels은 ReplicaSet에 대한 필터로 제공된다.
    - selector 섹션에서 matchLabels 필터를 사용했고, 파드를 생성할 때도 동일한 label을 사용했다.
    - 이렇게 하면 ReplicaSet은 모니터링할 파드를 알 수 있다.
    
    ❓ **이미 만들어진 Pod가 있는 경우에는?**
    
    - 이미 생성된 3개의 기존 파드가 있는 경우, 그 파드가 항상 3개가 실행되는 것을 모니터링하기 위해 ReplicaSet을 생성해야 한다.
    - 해당 레이블이 있는 파드가 3개 이미 존재하기 때문에 Pod에 새 인스턴스가 배포되지는 않는다.
    - 하지만, ReplicatSet spec에 template을 정의해야 한다.
    - 그 이유는 파드 중 하나가 죽었을 경우 ReplicaSet이 파드 수를 유지하기 위해 새 파드를 만들어야하기 때문이다.
    

## How to scale ReplicaSet

- ReplicaSet을 확장하는 방법에는 세 가지 방법이 있다.

### 1 ) definition file 수정

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata: 
	name: myapp-replicaset
	labels:
		app: myapp
		type: front-end
spec:
	template:
		metadata:
			name: myapp-pod
			labels:
				app: myapp
				type: front-end
		spec:
			containers:
			- name: nginx-pod
				image: nginx
	replicas: 6
	selector:
		matchLabels:
			type: front-end
```

```yaml
kubectl apply -f replicaset-definition.yaml
```

### 2 ) kubectl scale 커맨드

```yaml
kubectl scale --replicas=6 -f replicaset-definition.yaml
```

### 3 ) type과 name을 사용한 kubectl scale 커맨드

```yaml
kubectl scale --replicas=6 replicaset myapp-replicaset
```

### 4 ) 부하에 따른 자동 확장 옵션

---

# Deployment

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fa357cd5-7e1e-45f8-bcd6-2d149dfe5552/Untitled.png)

- 배포해야 할 웹 서버가 있다. 웹 서버의 인스턴스가 많이 필요한 상황이다.
- 그리고 애플리케이션의 새로운 버전이 나올때마다 Docker 인스턴스를 원활하게 업그레이드하고 싶다.
- 이때 모든 인스턴스를 한번에 업그레이드할 경우 유저에게 영향을 미칠 수 있기 때문에 하나씩 업그레이드하고 싶다.
- 이러한 업그레이드를 **롤링 업데이트**라고 한다.
- 만약 업그레이드를 하다가 오류가 발생하면, **최근 변경 사항들을 롤백**해야 한다.
- 이러한 기능을 Kubernetes Deployment로 사용할 수 있다.
- 애플리케이션이 담긴 컨테이너는 파드에 캡슐화된다. 파드를 여러개 보장해주는 역할을 ReplicaSet을 사용했다. Deployment는 ReplicaSet보다 한단계 더 높은 계층 구조에 있다.
- Deployment를 생성하면 ReplicaSet이 자동으로 생성된다.
- Deployment는 원활한 업그레이드를 위해 롤링 업데이트 사용, 롤백, 중지, 재개하는 기능을 제공한다.

### Definition of Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
	name: myapp-deployment
	labels:
		app: myapp
		type: front-end
spec:
	template:
		metadata:
			name: myapp-pod
			labels:
				app: myapp
				type: front-end
		spec:
			containers:
			- name: nginx-pod
				image: nginx
		replicas: 3
		selector:
			matchLabels:
				type: front-end
```

```yaml
kubectl apply -f deployment-definition.yaml

kubectl get deployments
```

Deployment는 ReplicaSet을 자동으로 만들기 때문에 ReplicaSet 또한 확인 가능하다.

```yaml
kubectl get replicaset
```

모든 오브젝트를 한 번에 보고 싶다면,

```yaml
kubectl get all
```

```yaml
kubectl create deployment --image=nginx nginx --dry-run=client o yaml > nginx-deploy.yaml
```

---

# Services

- 서비스는 내부 컴포넌트끼리, 애플리케이션 외부와 통신할 수 있게 해준다.
- 서비스는 애플리케이션을 다른 애플리케이션이나 사용자와 연결하는데 도움을 준다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f4c30001-2d0b-4ba8-8c0d-da7006048018/Untitled.png)

- 다양한 섹션을 실행하는 파드 그룹이 있다.
- 프론트엔드를 유저에게 서빙하는 그룹, 백엔드 프로세스를 실행하는 그룹, 외부 데이터 소스에 연결하는 그룹이 있다.
- Service는 이런 파드 그룹 간 연결을 가능하게 해준다.
    - 프론트엔드 애플리케이션을 엔드 유저가 사용할 수 있게 하고,
    - 백엔드와 프론트엔드 파드 간 통신을 돕고
    - 외부 데이터 소스 연결 설정에 도움을 준다.
- **서비스는 애플리케이션의 마이크로서비스 간 loose coupling을 가능하게 한다.**

### External Communication

- 서비스의 사용 사례를 보자.
- 웹 애플리케이션이 실행되고 있는 파드를 배포했다.
- 어떻게 외부 사용자가 웹 페이지에 접속할 수 있을까?

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8a1abc3f-4238-4de9-a6ae-f9bbfcae0fee/Untitled.png)

- 기존 설정은 Kubernetes 노드에는 IP 주소가 있다. 지금은 192.168.1.2이다.
- 내 노트북도 같은 네트워크에 있고, IP주소는 192.168.1.10이다.
- 그리고 내부 파드 네트워크 범위는 10.244.0.0이다. 현재 파드의 IP는 10244.0.2이다.

- 따라서 나는 파드와 다른 네트워크 대역에 있기 때문에 ping 하거나 접속할 수 없다.
- 그러면 어떻게 웹 페이지를 볼 수 있는 것일까?
- 먼저 쿠버네티스 노드에 SSH 연결하면 curl을 수행하여 파드의 웹 페이지에 접속할 수 있다. 또는 노드에 GUI가 있다면 브라우저를 열어서 [http://10.244.0.2를](http://10.244.0.2를) 입력하여 웹페이지를 볼 수 있다.
- 그러나 이것은 쿠버네티스 노드 내부에서 접근하는 방법이다. 우리가 정말로 원하는 것은 SSH 연결없이 내 노트북에서 웹 서버에 접속하는 것이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/556b537f-8405-4bcc-a7ea-e240d03e048a/Untitled.png)

- 노트북에서 SSH 연결 없이 쿠버네티스 노드의 IP에 접근하고, 웹 서버에 접속하고 싶다.
- 따라서 **노트북에서 노드로, 노드에서 파드로 중간에 request를 매핑할 무언가가 필요하다. 이 역할을 하는 것이 쿠버네티스 Service**다.
- 파드, ReplicaSet, Deployment와 마찬가지로 Kubernetes Service는 Object이다.

## Service Type

쿠버네티스의 서비스에는 3가지 타입이 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/34620fd2-903c-4ad9-b0c5-65a86fe8510a/Untitled.png)

### NodePort

노드와 노드 내 파드에서 내부 파드에 접근 가능하도록 하는 서비스이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/efae3a59-bf00-4251-8eb4-d19413d9945e/Untitled.png)

- 3개의 포트가 있다.
- 실제 웹 서버가 있는 파드의 포트는 80이며, Service가 request를 전달할 곳이기 때문에 TargetPort라고 한다.
- 두번째 포트는 서비스 자체 포트이다.
    - 그냥 Port라고 부른다.
    - **서비스는 노드 내 가상 서버와 같다.** 클러스터 내부에는 자체 IP 주소가 있는데, IP주소를 서비스의 ClusterIP라고 부른다. (10.106.1.12)
- 마지막으로 노드 자체 포트가 있다. 외부에서 웹 서버에 액세스하는 사용할 노드 포트이며, 지금은 30008로 설정되어 있다. 노드 포트는 30000에서 32767 사이의 범위에서 값을 가진다.

### create service

- apiVersion: v1
- kind: Service
- metadata:
    - name:
    - labels:
- spec:
    - type:
    - ports:
        - targetPort:
        - port:
        - nodePort: (30000 ~ 32767 사이에서 사용 가능한 포트 자동 배정)
    - selector:
        - labels와 selector를 사용해서 **서비스를 파드에 연**결해야 한다.
        - 연결할 파드의 labels 정보를 입력하면 된다.

```yaml
apiVersion: v1
kind: Service
metadata:
	name: myapp-svc
spec:
	type: NodePort
	ports:
	- targetPort: 80
		port: 80
		nodePort: 30008
	selector:
		app: myapp
		type: front-end
```

```yaml
kubectl apply -f service-definition.yaml

kubectl get svc
curl http://192.168.1.2:30008
```

- 지금까지 단일 파드에 매핑된 서비스에 대해 이야기했다.
- 프로덕션 환경에서 실행중인 웹 애플리케이션의 여러 인스턴스가 있을 것이고, 파드도 여러개 있을 것이다.
- 다행히 파드들은 같은 label을 가지고 있을 것이며, 서비스의 selector에서 해당 label을 가지고 있는 파드를 모두 선택하게 된다.
- 그 중 request를 전달할 파드는 파드 간 로드 균형을 조정하면서 랜덤 알고리즘으로 선택하게 된다.
- 따라서 **서비스는 파드 간 부하를 분산하면서 기본적으로 로드밸런서 역할을 하게 된다.**

❓ **만약 파드가 여러 노드에 분산되어 있다면 어떻게 될까?**

우리가 서비스를 만들 때 쿠버네티스는 자동으로 클러스터의 모든 노드에서 대상 포트를 매핑하도록 만든다. 이때 **클러스터의 모든 노드는 동일한 포트 번호를 사용**한다.
서비스는 Pod가 생성되거나 제거되면 자동으로 업데이트되고, 매우 유연하고 적응력있게 합니다.
