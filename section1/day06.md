# Day06

# Node Affinity vs Taints and Tolerations

- 3개의 노드와 3개의 파드가 있다고 가정해보자.
- 각각의 파드를 색과 일치하는 노드에 배치하고자 한다.

### Taints and Toleration

이를 위해 **taint와 toleration**을 사용해보자.

- taint를 이용하여 각각의 노드에 마크를 할 수 있다.
- 각각의 파드에 toleration을 설정하여, 노드의 taint를 견딜 수 있게 설정한다.
- 파드가 생성되면 노드는 각각의 노드에 알맞은 toleration이 있는지 확인하고, 있다면 이를 수락해준다.
- 하지만 taint와 toleration은 파드가 항상 원하는 노드로만 배치되는 것을 보장해주지 않는다.
    - taint나 toleration 세트가 없는 노드에 배치될 수도 있다.

### Node Affinity

Node Affinity를 사용하여 동일한 문제를 해결해보자.

- Node Affinity는 노드에 레이블을 지정한다.
- 그런 다음 각각의 파드에 node selector를 설정한다.
- 결과적으로, 파드를 노드에 연결해주고, 파드는 올바른 노드에 배치된다.
- 그러나 이것 또한 각 노드에 내가 원하는 파드 하나만 배치되기를 보장해주지 않는다.
    - 왜냐하면 node selector 설정을 안 한 파드가 해당 노드에 배치될 수 있기 때문이다.

따라서, taint와 toleration을 조합해서 함께 사용할 수 있다.

이를 통해 특정 파드에 대한 노드를 완전히 적용시킬 수 있다.

- 다른 파드를 방지하기 위해 먼저, taint와 toleration을 사용하고
- 이후 node affinity를 이용하여 노드에 배치되지 않도록 파드를 방지한다.

---

# Resources Limits

- 3개의 노드로 구성된 쿠버네티스 클러스터를 보자.
- 각 노드에는 사용 가능한 CPU, 메모리, 디스크 세트가 있다.
- 모든 파드는 해당 리소스 세트를 사용할 수 있는만큼 사용한다.

- 쿠버네티스 스케줄러는 파드가 이동하는 노드를 결정한다.
- 이때 스케줄러는 포드에 필요한 리소스 양과 노드에서 사용 가능한지를 고려한다.
- 만약 노드에 리소스가 충분하지 않다면 해당 노드에 파드를 배치하지 않고, 리소스가 충분한 노드에 파드를 배치시킨다.
- 만약 어느 노드에도 파드의 리소스를 수용할 수 없다면, 해당 파드는 Pending 상태가 된다. (이벤트를 보면 이유를 알 수 있다.)

### pod resource requests

- 각 파드의 리소스 요구 사항에 대해 살펴보자.
- 기본적으로 쿠버네티스 파드 또는 파드 안에 있는 컨테이너는  0.5 CPU와 256mb의 메모리가 필요하다.
- 이것은 컨테이너에 대한 리소스 요청양이다.
- 스케줄러가 파드를 노드에 배치할 때, 이 숫자를 사용하여 노드를 식별한다. 그리고 해당 노드는 충분한 양의 리소스를 사용할 수 있다.
- 이제 응용 프로그램에 더 많은 것이 필요하다는 것을 알고 있다면, 이 보다 더 많은 값을 파드 정의 파일에서 수정할 수 있다.
    - 요청을 추가하는 리소스 섹션을 추가하면 된다. 그리고 메모리, CPU에 대한 새로운 값을 지정한다. 이 경우 1GB의 메모리로 설정했다.

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp-color
	labels:
		name: simple-webapp-color
spec:
	containers:
	- name: simple-webapp-color
		image: simple-webapp-color
		ports:
			- containerPort: 8080
		resources:
			requests:
				memory: "1Gi"
				cpu: 1

```

- CPU 한 개의 의미에 대해 알아보자.
    - 0.1 CPU를 100m으로 표현하기도 한다.
    - 1m까지 내려갈 수 있지만 이 보다는 낮을 수 없다.
    - CPU 1개는 vCPU 1개와 동일하다.
        - 이는 AWS의 vCPU 하나 또는 GCP/Azure 코어 하나 혹은 스레드 1개와 같다.
    - 노드에 충분한 리소스가 있는 경우 컨테이너에 더 많은 수의 CPU를 요청할 수 있다.
- Memory에 대해 알아보자.
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4a752351-9fbd-4764-90b9-66ee69333bc6/Untitled.png)
    
    - 메모리도 256Mi를 지정할 수 있다.
    - Mi 접미사나 기가바이트에 대한 접미사 G를 사용할 수 있다.
    - G는 1000M를 의미하고, Gi는 1024Mi를 의미한다.
- 도커 세계에서 도커 컨테이너는 노드에서 사용할 수 있는 리소스 제한이 없다.
- 컨테이너가 노드에서 하나의 vCPU로 시작한다고 가정해보자.
    - 이것은 노드에 있는 기본 프로세스나 다른 리소스 컨테이너를 질식시키는 데 필요한 만큼의 리소스를 사용할 수 있다.
    - 따라서 파드의 자원 사용량에 대해 limit을 설정할 수 있다.
        - 디폴트로 쿠버네티스는 컨테이너 노드에서 하나의 vCPU만 사용하도록 제한된다.
        - 또한 메모리도 512Mi 제한을 기본적으로 제한되어 있다.
    - 기본 제한이 마음에 들지 않는다면, 파드 정의 파일에서 resources 섹션 아래 limits 섹션을 추가하여 변경할 수 있다.
    - 파드가 생성되면 쿠버네티스는 컨테이너에 대한 새로운 제한을 설정한다.
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
    	name: simple-webapp-color
    	labels:
    		name: simple-webapp-color
    spec:
    	containers:
    	- name: simple-webapp-color
    		image: simple-webapp-color
    		ports:
    			- containerPort: 8080
    		resources:
    			requests:
    				memory: "1Gi"
    				cpu: 1
    			limits:
    				memory: "2Gi"
    				cpu: 2
    ```
    

### Exceed Limits

- 만약에 Pod가 지정된 제한을 초과하는 리소스를 초과한다면 어떻게 될까?
- CPU의 경우 쿠버네티스는 지정된 제한을 초과하지 않도록 CPU를 조절한다.
    - 따라서 컨테이너는 limits 보다 더 많은 CPU 사용이 불가능하다.
- 그러나 메모리의 경우는 아니다.
    - 컨테이너는 한도보다 더 많은 메모리 리소스를 사용할 수 있다.
    - 따라서 파드가 지속적으로 한도보다 많은 메모리를 사용하려고 하면 파드가 종료될 수 있다.

---

# Daemon Sets

- DaemonSet은 ReplicaSet과 같다.
- 파드의 여러 인스턴스를 배포하는데 도움이 된다.
- 클러스터의 각 노드에서 하나의 파드만 실행한다.
- 클러스터에 노드가 추가될 때 자동으로 추가되고, 노드가 제거되면 자동으로 제거된다.
- DaemonSet은 파드의 사본 하나를 클러스터의 모든 노드에 항상 존재한다.

- DaemonSet 사용 사례
    - 모니터링 에이전트를 배포하고 싶거나
    - 각 노드의 로그 수집기
    - 클러스터의 변경 사항이 있는 경우 DaemonSet이 이를 처리할 것이다.
    - kube-proxy 구성 요소를 DaemonSet 형태로 배포해도 된다.
    - 네트워킹 솔루션 역시 DaemonSet 형태로 배포하면 좋다.

### Create DaemonSet Definition

ReplicaSet과 매우 유사한 방법으로 만들 수 있다.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
	name: monitoring-daemon
	labels:
		app: nginx
spec:
	selector:
		matchLabels:
			app: monitoring-agent
	template:
		metadata:
			labels:
				app: monitoring-agent
		spec:
			containers:
			- name: monitoring-agent
				image: monitoring-agent
```

### View DaemonSets

```yaml
kubectl get daemonsets
```

---

# Static PODs

- Kubelet은 노드를 독립적으로 관리할 수 있다.
- Kubelet이 혼자서 할 수 있는 것은 Pod 생성이다.
- Kube API 서버 없이 kubelet에 파드를 읽도록 kubelet을 구성할 수 있다.
- /etc/kubernetes/manifests 디렉토리에 파드 정의 파일을 배치시키면 된다.
- Kubelet은 주기적으로 이 디렉터리에서 파일을 확인한다.
    - 이러한 파일을 읽고 호스트에 파드를 생성한다.
    - 그리고 파드가 계속 살아있고, 죽으면 다시 시작하려고 시도한다.
    - 파일을 변경하면 kubelet은 파드를 다시 생성한다.
    - 또한 파일 삭제하면 파드도 자동으로 삭제된다.

### static Pod

이처럼 Kube API 개입 없이 kubelet 혼자서 자체 생성한 파드를 정적 파드라고 한다.

- 오직 파드만 생성할 수 있다.
    - Replica Set이나 Deployment, Service는 생성할 수 없다.
    - kubelet은 오직 파드 레벨이기 때문에 파드만 생성할 수 있다.

### Path

- 호스트의 모든 디렉토리는 kubelet에 전달된다.
- kubelet의 옵션은 /etc/kubernetes/manifests 경로이다.
- 또는 kubelet 서비스 옵션에 디렉토리 경로를 지정할 수 있다.

- kubelet 서비스 파일에서 옵션을 체크하고
- 없으면 구성 옵션을 찾으면 된다.
- 정적 파드가 생성되면 docker ps 명령을 실행해서 볼 수 있다.
    - 왜? kube 제어 명령어가 아닐까?
        - 아직 쿠버네티스 클러스터가 없기 때문이다.

### Use Case

- 쿠버네티스 마스터 노드를 정적 포드를 이용하여 배포한다.
- kubelet을 먼저 설치하는 것부터 시작한다.

### Static PODs vs DaemonSets

- DaemonSets
    - 응용 프로그램의 한 인스턴스를 보장하는데 사용된다.
    - 클러스터의 모든 노드에서 사용 가능하다.
    - Kube API 서버를 통해 데몬 세트 컨트롤러에 의해 처리된다.
- Kubelet
    - Kube API 서버 간섭 없이 정적 포드를 이용해 배포할 수 있다.
    - 컨트롤 플레인 구성 요소 자체를 배포한다.
