# Day05

노드에 파드를 수동으로 스케줄링 하는 방법이다.

## Manual Scheduling

### How Scheduling Works

- 클러스터에 스케줄러가 없으면 어떻게 할까?
    - 내장된 스케줄러에 의존하지 않고 직접 파드를 예약하려고 한다.
- 이때 스케줄러는 백엔드에서 어떻게 작동할까?
    - 간단한 Pod definition file부터 시작해보자.
    - 모든 파드에는 Node Name이라는 필드가 있다. default는 값으로 들어가 있지 않다.
    - 일반적으로 definition file을 만들 때 지정하지 않고, 쿠버네티스가 자동으로 추가하는 필드이다.
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9a821729-26a4-4f09-b954-80887a06d1c5/Untitled.png)
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/39d14f89-57dd-482a-8aaf-53077d11e912/Untitled.png)
    
- 스케줄러는 모든 파드들을 검사하면서 **Node Name 속성이 설정되지 않은 파드**를 찾는다.
- 이들이 바로 **스케줄링 후보**이다.
- 다음, **스케줄링 알고리즘을 사용하여 파드에 적합한 노드를 고른다.**
- **골랐으면 파드를 그 노드에 스케줄링한다.**
- 이때 **바인딩 오브젝트를 생성**하면서 **파드의 Node Name에 그 노드의 이름이 들어가게 된다.**

## No Scheduler

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d0e752e9-fca0-4427-a431-04d10b8e99fd/Untitled.png)

- 만약 모니터링하고 스케줄링할 스케줄러가 없다면 어떻게 해야 할까?
- 이 상태에서 파드는 계속 보류중(pending) 상태가 된다.
- 이를 해결하려면 수동으로 파드를 노드에 할당해야 한다.
- 스케줄러 없이 파드를 스케줄링하는 가장 쉬운 방법은 생성할 때 pod specification file에서 Node Name에 노드를 지정하는 것이다.
    - 이렇게 하면 파드가 지정된 노드에 할당된다.
- Node Name에 노드를 지정하는 것은 파드를 만들 때만 가능하다.

❓ **이미 파드가 생성된 경우에 파드를 노드에 할당하고 싶다면 어떻게 해야할까?**

- 쿠버네티스는 파드의 Node Name 속성을 수정하지 못하게 해놓았다.
- 이럴 때는 바인딩 오브젝트를 만들고, 파드의 바인딩 API에 POST request를 보내면 된다.
- 이 방식은 실제로 스케줄러가 수행하는 작업을 따라한 것이다.

- 아래와 같이 바인딩 오브젝트를 만든다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5fbe901b-a78c-4327-806f-32c8e35cb8ce/Untitled.png)

- 바인딩 객체에서 target의 하위 name에 노드 이름이 들어간다.

```yaml
apiVersion: v1
kind: Binding
metadata:
	name: nginx
target:
	apiVersion: v1
	kind: Node
	name: node02
```

- 그런 다음, **바인딩 API에 POST request를 보낸다.**
    - request를 보낼 때에는, YAML과 동일한 내용을 JSON으로 변환한 데이터를 사용해야 한다.

```yaml
curl --header "Content-Type:application/json" --request POST --data '{"apiVersion":"v1", "kind":"Binding", ...}
http://$SERVER/api/v1/namespace/default/pods/$PODNAME/binding/
```

---

# Labels and Selectors

Labels와 Selector은 그룹화하는 표준 방법이다.

## Labels

- 다양한 종의 동물 세트를 다양한 기준에 따라 그룹화 및 필터링할 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/50a07e75-ba05-40a1-9087-bf76d5c76b8c/Untitled.png)

- Labels는 각 항목에 첨부된 속성이므로 클래스, 종류 및 색상에 대한 속성을 각 항목에 추가된다.

## Selector

- 이러한 항목을 필터링하는데 도움이 된다.
- 사용자가 올바른 콘텐츠를 필터링하고 찾는데 도움이 되는 태그를 지정하는 것과 같이 사용된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c8c6c9b6-c385-4018-ab93-4b3e283615a6/Untitled.png)

## Labels & Selectors in Kubernetes

- 쿠버네티스의 경우 Pod, ReplicaSet, Deploy, Service이 서로 다른 오브젝트이다.
- 시간이 지남에 따라 클러스터에 오브젝트는 수백 또는 수천개가 될 수 있다.
- 유형별로 오브젝트를 그룹화하거나 애플리케이션 또는 기능별로 개체를 보는 것과 같이 다양한 범주별로 다양한 개체를 필터링하고 볼 수 있는 방법이 필요하다.
- 그것이 무엇이든 Label과 Selector를 사용하여 오브젝트를 각 오브젝트에 대해 애플리케이션이나 기능에 따라 레이블을 부착하여 그룹화하고 선택할 수 있다.
- 선택하는 동안 특정 오브젝트를 필터링할 조건을 지정한다.
    - 예를 들어 app=App1과 같이다.

### Labels

- Label을 어떻게 지정할까?
- Pod Definition file에서 metadata에서 labels이라는 섹션을 작성한다.
- 그 아래에 다음과 같은 키 값 형식으로 labels를 추가한다.
- 원하는 만큼 레이블을 추가할 수 있다.

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp
	labels:
		app: App1
		function: Front-end
spec:
	containers:
	- name: simple-webapp
		image: simple-webapp
		ports:
		- containerPort: 8080
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/850110b6-d2d7-49a9-af70-44c8a4f1763f/Untitled.png)

### Selector

- 파드가 생성되면 레이블이 있는 파드를 선택하려면 selector 옵션과 함께 kubectl get pods 커맨드를 사용하고 app=App1과 같은 조건을 지정한다.

```yaml
kubectl get pods --selector app=App1
```

- 이것은 레이블 및 selector의 사용 사례 중 하나이다.
- 쿠버네티스 오브젝트는 내부적으로 레이블과 selector을 사용하여 서로 다른 오브젝트를 함께 연결한다.

### ReplicaSet

- 예를 들어, 3개의 서로 다른 파드로 구성된 ReplicaSet를 생성하려면 먼저 파드 정의에 레이블을 지정하고 ReplicaSet에서 selector를 사용하여 파드를 그룹화한다.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
	name: simple-webapp
	labels:
		app: App1
		function: Front-end
spec:
	replicas: 3
	selector:
		matchLabels:
			app: App1
	template:
		metadata:
			labels: 
				app: App1
				funtion: Front-end
		spec:
			containers:
			- name: simple-webapp
				image: simple-webapp
```

- ReplicaSet Definition 파일에서 두 위치에 정의된 레이블을 볼 수 있다.
- template 섹션 아래에 정의된 레이블은 파드에 구성된 레이블이다.
- 상단에 표시된 레이블은 ReplicaSet 자체의 레이블이다.
- ReplicaSet가 Pod를 검색하고 있기 때문에 지금은 ReplicaSet의 레이블에 대해 크게 걱정하지 않는다.
- ,ReplicaSet의 레이블은 ReplicaSet를 검색하는 다른 오브젝트를 구성하는데 사용한다.
- ReplicaSet를 파드에 연결하기 위해 파드에 정의된 레이블과 일치하도록 ReplicaSet spec 아래의 selector 필드를 구성한다.
    - 단일 label로 매칭된다면 하나만 사용해도 된다.
    - 그러나 label은 같지만 기능이 다른 Pod가 있을 수 있다고 생각되면 두 레이블을 지정하여 ReplicaSet에서 올바른 부분이 발견되도록 할 수 있다.
- ReplicaSet 생성시 레이블이 일치하면 ReplicaSet이 성공적으로 생성된다.

### Service

- 서비스와 같은 경우 다른 오브젝트와 동일하게 작동한다.
- 서비스가 생성되면 definition 파일에 정의된 selector을 사용하여 ReplicaSet에 설정된 레이블을 일치시킨다.

```yaml
apiVersion: v1
kind: Service
metadata:
	name: my-service
spec:
	selector:
		app: App1
	ports:
	- protocol: TCP
		port: 80
		targetPort: 9376
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/68a0ba1f-078e-40b6-b817-63babf2bef28/Untitled.png)

### Annotations

- 주석을 살펴보자.
- 레이블과 selector은 오브젝트를 그룹화하고 선택하는데 사용되는 반면, **주석은 정보 제공 목적으로 기타 세부 정보를 기록하는데 사용된다.**
- 예를 들어, 이름/버전/빌드정보 등과 같은 도구 세부 정보 또는 일종의 통합 목적으로 사용될 수 있는 연락처 세부 정보, 전화 번호, 이메일 ID 등이 있다.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
	name: simple-webapp
	labels:
		app: App1
		function: Front-end
	annotations:
		buildversion: 1.34
spec:
	replicas: 3
	selector:
		matchLabels:
			app: App1
	template:
		metadata:
			labels: 
				app: App1
				function: Front-end
		spec:
			containers:
			- name: simple-webapp
				image: simple-webapp
```

---

# Taints and Tolerations

Pod와 노등 관계에 대해 이야기해보자.

- 사람은 노드이고 벌레는 파드라고 가정해보자.
- taint와 tolerations는 클러스터에 대한 보안 또는 침입과 아무 상관이 없다.
- taint 및 tolerations는 어떤 파드가 어떤 노드에 스케줄될 수 있는지 제한을 설정하는데 사용된다.

가정해봅시다.

- 세 개의 워커 노드가 있는 간단한 클러스터가 있다.
- 노드의 이름은 1, 2, 3이고, 파드는 A, B, C, D라고 해보자.
- 파드가 생성되면 쿠버네티스 스케줄러는 이러한 파드를 사용 가능한 워커노드에 배치하려고 시도한다.
- 현재로써는 아무런 제한이 없기 때문에 스케줄러는 모든 노드에 균등하게 균형을 맞추어 파드를 배치한다.

- 노드 1에 특정 use case나 애플리케이셔을 위한 전용 리소스가 있다고 가정하자.
- 그렇다면 노드1에는 특정 애플리케이션이 속하는 파드만 배치하고 싶다.
- 그러기 위해서는 두가지 요구사항을 만족시켜야 한다.
    - 노드1에 원치않은 파드를 배치하지 않게 하고,
    - 노드1에 특정한 파드만 배치해야 한다.
- 첫째는 해당 **노드에 taint를 배치하여 모든 파드가 그 노드에 배치되는 것을 방지**한다.
    - 예를 들어, taint는 blue라고 하자.
- 파드는 디폴트로 toleration을 가지고 있지 않다. 따라 명시해주지 않을 경우, 어떠한 파드도 taint가 있는 곳으로 갈 수 없다.
- 따라서 현재 어떤 파드도 taint blue를 참을 수 없기 때문에 노드 1에는 어떤 파드도 배치될 수 없다.

- 특정한 파드를 해당 노드에 배치시켜야 한다.
- 어떤 파드가 특정한 taint에 tolerant한지(참을 수 있는지) 지정해야 한다.
- 파드 D만 노드 1에 배치되는 것을 허용하고 싶다.
- 그렇다면 파드 D에 toleration을 추가한다.
- 이제 파드 D는 blue taint를 참을 수 있게 된다. 따라서 스케줄러가 이 파드를 노드1에 배치하려고 할 때, 배치가 가능해진다.
- 노드 1은 blue taint에 대한 toleration을 가지고 있는 파드만 허용할 수 있다.

이것이 taints와 tolerations가 존재할 때 파드가 스케줄되는 방식이다.

- 스케줄러는 파드 A를 노드 1에 배치하려고 하지만 노드1의 taint에 의해 허용되지 않고, 노드 2로 이동한다. (파드 C도 마찬가지)
- 그리고 스케줄러는 파드 B를 노드 1에 배치하려고 시도하고, 허용되지 않고 free 노드인 노드 3에 배치된다.
- 마지막으로 스케줄러는 파드 D를 노드 1에 배치하려고 했을 때, 파드 D는 노드 1에 대한 tolerations이 있으므로 허용된다.

**taint는 노드에 설정되고, toleration은 파드에 설정**된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e63656e3-cdae-4e59-a93a-91487e59b67a/Untitled.png)

## Taint

- kubectl taint nodes를 커맨드를 사용하여 노드에 taint를 설정한다.
- 커맨드 뒤에 key-value 쌍으로 노드 이름과 taint를 지정하면 된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d18cc114-91ba-47cc-921b-ba19db9c35d4/Untitled.png)

```yaml
kubectl taint nodes <node-name> key=value:taint-effect
```

### Example

```yaml
$ kubectl taint nodes node1 app=blue:NoSchedule
```

- 노드를 blue 애플리케이션 전용으로 지정하려는 경우, key-value 쌍은 app=blue가 된다.
- taint effect는 taint에 toleration가 없는 파드를 어떻게 처리할 것인지를 결정한다.
- taint effect에는 3가지 종류가 있다.
    - NoSchedule: **toleration이 없는 파드는 taint가 있는 노드에 스케줄링 되지 않는다.**
    - PreferNoSchedule: 시스템이 되도록이면 taint가 있는 노드에 파드를 배치하지 않으려하지만 가능은 하다.
    - NoExcuse: toleration이 없는 새로운 파드는 노드에 배치되지 않으며, **이미 노드에 존재하는 파드는 toleration가 없다면 축출**된다.

## Toleration

- 파드에 toleration을 추가하려면 먼저 파드 definition file을 가져와야 한다.
- definition file의 spec 섹션에 tolerations라는 섹션을 추가한다.
- taint를 생성했을 때 사용한 값과 동일하게 입력한다.
- value는 blue이고, effect는 NoSchedule이다.
- **tolerations 섹션의 모든 값은 쌍따옴표로 감싸야 한다.**

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: myapp-pod
spec:
	containers:
	- name: nginx-container
		image: nginx
	tolerations:
	- key: "app"
		operator: "Equal"
		value: "blue"
		effect: "NoSchedule"
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d0e4768c-2042-40ea-90f2-d5e472fc87aa/Untitled.png)

- 이제 파드가 tolerations를 가지고 생성되거나 toleration을 가지도록 업데이트될 때, effect에 따라 스케줄링이 안되거나 기존 노드에서 제거될 것이다.

## Taint - NoExcuse

- taint effect를 NoExcuse로 설정하면 이미 해당 노드에 있던 파드는 제거된다.
- 제거된 파드는 죽는다.

- taints와 tolerations은 노드가 특정한 파드를 허용하는 것에 대한 제한이다.
- 즉, 특정 파드가 특정 노드로 가는 것이 아니라, 노드가 특정 toleration을 가지고 있는 파드만 허용한다는 의미이다.
    - 예를 들어, 파드 D가 항상 노드 1에 있는 것을 보장하지 않는다.
    - 왜냐하면 다른 노드에 taint가 적용되지 않았기 때문에 파드 D는 다른 노드에 있을 수 있기 때문이다.

- 만약 특정 파드가 특정 노드로 가는 것을 제한하고 싶다면 node Affinity를 사용하면 된다.

- 마스터 노드는 파드를 호스팅하고 모든 관리 소프트웨어를 실행할 수 있다.
- 이제 스케줄러가 마스터 노드에 파드를 스케줄링하지 않는다는 것을 알 수 있다.
- 왜냐하면, **쿠버네티스 클러스터가 처음 설정되면 마스터 노드에 taint가 자동으로 설정되어 파드가 마스터 노드에 스케줄링되지 않도록 하기 때문이다.**
- 우리는 이것을 확인할 수 있고, 필요에 따라 동작을 수정할 수 있다.
- 그러나 최선은 마스터 노드에 애플리케이션 워크로드를 배포하지 않는 것이다.

```yaml
kubectl describe node kubemaster | grep Taint
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/25a912b9-e1ba-4701-9aa8-f9ad572bc02c/Untitled.png)

---

# Node Selectors

- 가정해보자.
- 3개의 노드가 있는 클러스터가 있으며 그 중 2개는 더 낮은 하드웨어 리소스를 가진 더 작은 노드이다. 다른 하나는 더 높은 리소스로 구성된 더 큰 노드이다.
- 다양한 종류의 워크로드가 클러스터에서 실행 중이다.
- 더 큰 노드를 더 높은 마력이 필요한 데이터 처리 워크로드를 전용으로 사용하려고 한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9488f06e-9c3c-48da-ac84-197fd4ec377b/Untitled.png)

- 그러나 현재 설정에서는 모든 파드는 모든 노드로 이동할 수 있다.
- 따라서 이 경우 파드 C가 노드 3에 배치되면 죽을 수 있다.
- 이를 해결하기 위해 파드가 특정 노드에서만 실행되도록 제한을 설정할 수 있다.

- 두 가지 방법이 있다.
    - 첫번째는 node selector을 이용하는 것이다.
- 이를 생성하기 위해 이전에 생성한 definition 파일을 보자.
- 이 파일에는 데이터 처리 이미지와 함께 파드를 생성하기 위한 간단한 정의가 있다.

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: myapp-pod
spec:
	containers:
	- name: data-processor
		image: data-processor
	nodeSelector:
		size: Large
```

- 파드가 더 큰 노드에서 실행되도록 제한하려면 spec 섹션에 nodeSelector라는 새 속성을 추가한다.
- size: Large를 입력한다. 이것은 key-value쌍의 노드 label이다.
- 스케줄러는 이 레이블을 사용하여 파드를 배치할 올바른 노드를 식별한다.
- 이처럼 Node Selector에서 레이블을 사용하려면 파드를 만들기 전 노드에 레이블을 지정해야 한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/49fe5656-0a42-4f51-ae51-2077f5c8504e/Untitled.png)

### 레이블 지정

- 노드에 레이블을 지정해보자.
- 노드에 레이블을 지정하려면 아래 포맷의 커맨드를 사용한다.

```yaml
kubectl label nodes <node-name> <label-key>=<label-value>
```

### 예시

```yaml
kubectl label nodes node-1 size=Large
```

이후 노드에 레이블을 지정했으므로 파드 생성을 한다.

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: myapp-pod
spec:
	containers:
	- name: data-processor
		image: data-processor
	nodeSelector:
		size: Large
```

```yaml
kubectl create -f pod-definition.yaml
```

파드가 생성되면 원하는대로 노드1에 배치된다.

### Node Selector - Limitations

- Node Selector는 우리의 목적에 부합했지만 한계가 있다.
- 우리는 단일 레이블과 selector를 사용했다.
- 그러나 만약 요구 사항이 훨씬 더 복잡하다면 어떨까?
    - 예를 들어, Large 노드나 Medium 노드에 파드를 배치하는 것과 같은 것이나
    - Small이 아닌 노드에 파드를 배치하는 것과 같은 상황을 말한다.
- Node Selector로 이를 달성할 수 없다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9798a0b4-85bc-43e9-9ecc-bd2360d0145f/Untitled.png)

---

# Node Affinity

Node Affinity는 파드가 특정한 노드에서 호스팅되는 것을 보장하는 것이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b3c5f2b0-cba7-46d0-918d-5f61bf2e194a/Untitled.png)

- 예를 들어 위와 같이 대용량 데이터 처리 파드를 노드 1에 배치되도록 해보자.
- 이 작업은 Node Selector를 통해 간단하게 해결할 수 있다.
- 그러나 Node Selector는
    - Large 노드 혹은 Medium 노드에 파드를 배치하는 것
    - Small이 아닌 노드에 파드를 배치하는 것
- 과 같은 advance expressions을 제공할 수 없다.

## Node Affinity

- Node Affinity는 advanced expressions를 제공한다.

### Node Affinity로 Large 노드 혹은 Medium 노드에 파드 배치하기

- 아래 Pod definition file은 위에서 구현한 NodeSelector를 이용한 정의 파일과 동일한 내용이다.

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: myapp-pod
spec:
	containers:
	- name: data-processor
		image: data-processor
	affinity:
		nodeAffinity:
			requiredDuringSchedulingIgnoredDuringExcution:
				nodeSelectorTerms:
				- matchExpresstions:
					- key: size
						operator: In
						values:
						- Large
						- Medium
```

### Node Affinity로 Small이 아닌 노드에 파드 배치하기

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: myapp-pod
spec:
	containers:
	- name: data-processor
		image: data-processor
	affinity:
		nodeAffinity:
			requiredDuringSchedulingIgnoredDuringExcution:
				nodeSelectorTerms:
				- matchExpresstions:
					- key: size
						operator: NotIn
						values:
						- Small
```

### Node Affinity로 Small이 아닌 노드에 파드 배치하기2

노드의 레이블을 Large와 Medium만 설정하고 Small은 레이블을 설정하지 않았다면, 레이블이 존재하는 노드에만 파드를 배치함으로써 파드를 Large 노드나 Medium 노드에 배치할 수 있다.

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: myapp-pod
spec:
	containers:
	- name: data-processor
		image: data-processor
	affinity:
		nodeAffinity:
			requiredDuringSchedulingIgnoredDuringExcution:
				nodeSelectorTerms:
				- matchExpresstions:
					- key: size
						operator: Exists
```

## Node Affinity Type

- Node Affinity와 Node가 일치하지 않는다면?
- Large 노드에 파드를 배치하려고 했는데 Large라는 레이블이 붙은 노드가 없는 경우, 파드가 이미 배치되었는데 누군가 노드의 레이블을 변경하는 경우가 있다.
- 이때 파드가 계속 노드에 남아있을까? 이러한 것들에 대한 설명은 긴 문장 같이 생긴 Node Affinity Type이 해준다.
- Node Affinity Type은 Node Affinity와 관련된 스케줄러 동작을 정의한다.

- Node Affinity Type은 두 가지이다.
    - required**DuringScheduling**Ignored**DuringExecution**
    - preferredDuringSchedulingIgnoredDuringExecution
- Planned Node Affinity Type
    - requiredDuringSchedulingRequiredDuringExecution
    - preferredDuringSchedulingRequiredDuringExecution

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cfe67a12-e7f5-4163-9a6f-e5598174f5d4/Untitled.png)

파드의 라이프사이클에는 Node Affinity에 대해 두 가지 상태가 있다. 

DuringScheduling과 DuringExcution이다.

### DuringScheduling

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/73352377-a57d-4851-ab49-836d7cc87ffa/Untitled.png)

- 파드가 이미 존재하지 않고 처음 생성된 상태이다.
- 파드가 처음 생성될 때 지정된 Node Affinity 규칙이 파드를 올바른 노드에 배치하는 것은 당연하다.
- 이제 일치하는 레이블이 있는 노드를 사용할 수 없으면 어떻게 될까?
    - 예를 들어, 노드에 Large 레이블을 지정하는 것을 잊은 경우이다.
- 여기에는 Node Affinity Type이 적용된다.
    - Type이 **Required을 선택하면 스케줄러는 지정된 Node Affinity을 사용하여 파드를 노드에 배치하도록 지시**한다.
        - 그리고 노드를 찾을 수 없으면 파드가 예약되지 않는다.
    - Type이 **preferred인 경우**에는,
        - 일치하지 않은 경우 스케줄러는 단순히 Node Affinity 규칙을 무시하고 사용 가능한 노드에 파드를 배치한다.

### During Execution

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/aee84886-11b4-43ea-8b7f-1bc453206684/Untitled.png)

- 파드가 실행 중이고, 노드 레이블 변경과 같이 Node Affinity도 영향을 미치는 환경이 발생한 상황이다.
    - 예를 들어 관리자가 size=Large 레이블을 삭제하는 경우
    - 노드에서 실행 중인 파드는 어떻게 될까?
- During Execution이 Ignored로 설정되어 있다.
    - 파드는 계속 실행되며 Node Affinity의 변경사항은 일단 예약되면 파드에 영향을 미치지 않는다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6edfd542-64cd-4194-859c-c52c81658d1d/Untitled.png)

Planned Node Affinity Type은 During Execution에 차이가 있다.

- **RequiredDuringExecution**은 Node Affinity을 충족하지 않는 노드에만 실행 중인 파드를 제거한다.
    - Large 노드에서 실행 중인 파드는 Large 레이블이 노드에서 제거되면 파드 또한 제거되거나 종료된다.
