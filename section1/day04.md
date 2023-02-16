# Day04

# Cluster IP

- 풀스택 웹 애플리케이션에는 일반적으로 다양한 종류의 파드가 있다. ( 프론트 엔드 파드, DB 파드 등 )
- 웹 프론트엔드 서버는 백엔드 서버와 통신해야하고, 백엔드 서버는 데이터베이스, redis 서버와 통신해야한다.

![image](https://user-images.githubusercontent.com/76610641/219311633-c1570559-4f3e-427a-9f36-81e262e80f69.png)

❓ **애플리케이셔 간 연결을 설정하는 올바른 방법은 무엇일까?**

- 모든 파드에 할당된 IP 주소가 있다.
- 파드는 얼마든지 죽을 수 있고, 새로운 파드로 생성된다.
- 따라서 **파드에 할당된 IP 주소는 정적 주소가 아니다.**
- 애플리케이션 간 통신을 한다고 했을 때, 이 파드 IP에 의존할 수 없다.
- 또한 프론트엔드가 백엔드와 연결하려고 할 때 세 개의 파드 중 어디로 가야되는지 결정해야 한다.
- 이와 같은 문제를 해결하기 위해 **쿠버네티스 서비스는 Pod를 그룹화하여 파드에 액세스하 수 있는 단일 인터페이스(Virtual IP)를 제공**한다.
- 예시
    - 백엔드 파드를 위한 서비스는 모든 백엔드 파드를 그룹화 시킬 것이다.
    - 그리고 다른 파드가 서비스에 액세스할 수 있도록 단일 인터페이스를 제공한다.
    - request는 서비스 밑에 있는 파드 중 랜덤하게 하나의 파드에게 향하게 된다.
- 이를 통해 우리는 쿠버네티스 클러스터에서 쉽게 마이크로서비스 기반 애플리케이션을 쉽게 배포할 수 있다.
- 다양한 서비스 간의 연결에 영향을 주지 않고, 각 레이어의 크기를 조정하거나 이동할 수 있다.
- 각 서비스에는 IP와 이름이 할당된다.
- **클러스터 내부에서 사용하는 이름으로, 이러한 타입의 서비스를 Cluster IP**라고 한다.

### Create ClusterIP

- apiVersion: v1
- kind: Service
- metadata:
    - name:
- spec:
    - types: ClusterIP
    - ports:
        - targetPort: 백엔드 노출 포트
        - port: 서비스 포트
    - selector: 서비스를 특정 파드 그룹에 연결하기 위한 것이다. 여기에는 labels가 들어간다.

```yaml
apiVersion: v1
kind: Service
metadata:
	name: back-end
spec:
	types: ClusterIP
	ports: 
	- targetPort: 80
		port: 80
	selector:
		app: myapp
		type: back-end
```

```yaml
kubectl create -f service.yaml

kubectl get svc
```

---

# LoadBalancer

![image](https://user-images.githubusercontent.com/76610641/219311783-4d335f03-d4aa-41d9-80fe-087f2ca41f8d.png)

- NodePort 서비스는 워커 노드 port에서 외부 애플리케이션을 사용할 수 있도록 해준다.

![image](https://user-images.githubusercontent.com/76610641/219311837-1d1190c1-1b56-4afd-a929-fa68f2609185.png)

- 프론트엔드 애플리케이션에 초점을 돌리겠다.
- 투표앱과 결과앱이 있다.
- 우리는 이런 파드가 클러스터의 워커 노드에서 호스팅된다는 것을 알고있다.
- 클러스터에 4개의 워커 노드가 있다고 가정해보자.
- 그리고 NodePort 타입의 서비스를 생성하여 외부 유저가 애플리케이션에 액세스할 수 있다고 해보자.
- NodePort 서비스는 노드로 들어오는 서비스를 수신하고, 각 노드에 트래픽을 라우팅하는데 도움을 준다.

**그런데 애플리케이션에 액세스할 때 유저에게 어떤 URL을 제공할까?**

- IP와 포트를 통해 두 애플리케이션은 모두 어디든 액세스할 수 있다.
- 따라서 제공할 수 있는 조합은 투표앱 4개 조합, 결과앱 4개 조합이다.
- 만약 두 개의 노드에서만 Pod가 호스팅 되는 경우, 여전히 클러스터에 있는 모든 노드의 IP에 액세스할 수 있다.
- 투표앱에 대한 파드가 IP 70, 71로 끝나는 노드에 배치되었다고 가정하자.
    - 파드들은 여전히 모든 노드에서 접근 가능하다.
- 이것이 서비스가 구성되는 방식이고, 이 URL을 공유하고 유저가 애플리케이션에 액세스할 수 있도록 한다.

💡 **하지만 엔드 유저가 정말 원하는 것은 아니다.**

- 엔드 유저가 원하는 것은 [http://example-vote.com 이나](http://example-vote.com이나) [http://example-result.com과](http://example-result.com과) 같은 단일 URL로 프로그램에 액세스하는 것이다.
- 이를 달성하는 방법은 로드밸런서용 새 VM을 생성하고 로드밸런서를 설치하고 구성하는 것이다.
- 적합한 로드 밸런서는 AJ/Proxy/nginx와 같은 것이 있다.
- 그 다음에 노드들에 트래픽을 라우팅하도록 로드밸런서를 구성한다.

![image](https://user-images.githubusercontent.com/76610641/219311888-4dde02f2-4106-4672-9fe5-78318764e659.png)

- 외부 로드밸런싱에 대한 설정을 모두 마쳤다.
- 유지 관리를 해야하는데 이 작업은 지루할 수 있다.
- 그러나 우리는 Google Cloud, AWS, Azure와 같은 클라우드 플랫폼에서 네이티브 로드 밸런서를 활용할 수 있다.
- 쿠버네티스 클라우드 공급자의 네이티브 로드밸런서를 활용하는 것을 지원한다. 따라서 우리는 서비스 타입만 설정하면 된다.

```yaml
apiVersion: v1
kind: Service
metadata:
	name: myapp-service
spec:
	type: LoadBalancer
	ports:
	- targetPort: 80
		port: 80
		nodePort: 30008
```

---

# Namespaces

- Namespace는 집이다.

![image](https://user-images.githubusercontent.com/76610641/219311932-25598f3d-e026-4795-909e-787b5f402d05.png)

- 우리는 하나의 Namespace 안에서 Pod, Service, Deployment와 같은 서비스를 만들었다.
- Namespace는 따로 설정하지 않으면 default Namespace라고 한다.
- default Namespace는 클러스터가 처음 설정될 때 쿠버네티스에 의해 자동으로 생성된다.
- 쿠버네티스는 파드 및 서비스 세트를 생성한다.

![image](https://user-images.githubusercontent.com/76610641/219311978-9d74a517-9b41-490a-a8a1-ffc8b1822635.png)

- 내부 목적으로, 네트워킹 솔루션에 필요한 것과 DNS 서비스 등은 다른 Namespace에 생성한다.
    - 이를 실수로 삭제하거나 수정하는 것을 방지하고, 유저로부터 격리하기 위함이다.
    - kube-system이라는 클러스터가 시작될 때 생성된다.
    - 이곳은 모든 사용자가 자원을 사용할 수 있어야 한다.
- 작은 환경은 Namespace에 대해 걱정할 필요가 없다.
- 그러나 클러스터를 확장하고 사용할 때와 마찬가지로 기업 또는 생산 목적으로 Namespace 사용을 고려할 수 있다.
    - 예를 들어 개발 및 프로덕션 환경 둘 다 동일한 클러스터를 사용하는 경우, 그들 사이의 리소스를 격리하려는 목적으로 각각에 대한 고유한 Namespace를 만들 수 있다.
    - 이렇게 하면 개발하는 동안 프로덕션 리소스를 실수로 수정하는 일이 없다.
- 각 Namespace는 누가 무엇을 할 수 있는지 정의하고, 리소스 할당량을 할당하는 고유한 정책을 가질 수 있다.
    - 리소스 사용량을 할당하면 각 네임스페이스가 허용 한도 이상의 리소스를 사용하지 않는다.
- 네임스페이스 내의 리소스는 단순히 이름으로 참조할 수 있다.
- 필요한 경우 웹 애플리케이션 파드 또한 다른 네임스페이스에 접근할 수 있다.
    - 이 경우 서비스 이름에 네임스페이스를 추가하면 된다.
    - default Namespace의 웹 파드의 경우, 개발 환경에서 데이터베이스를 연결하기 위해 servicename.namespace.svc.cluster.local 형식을 사용한다.
    - 즉, dbservice.dev.svc.cluster.local이 된다.
- 서비스가 생성될 때 DNS 항목은 이 형식을 자동으로 추가된다.
- 서비스의 DNS 이름을 자세히 보면 마지막 부분에 cluster.local이 있다.
    - cluster.local은 쿠버네티스 클러스터의 도메인 이름이다.
    - svc는 서비스의 하위 도메인이고,
    - dev는 namespace를 의미한다.
    - 이후 service 자체의 이름이 나온다.

### Namespace 운영

```yaml
kubectl get pods
```

는 default namespace에 있는 파드만 조회하는 것이다.

```yaml
kubectl get pods -n kube-system
```

이는 kube-system에 있는 파드를 조회하는 것이다.

![image](https://user-images.githubusercontent.com/76610641/219312023-5713c1fb-3d90-49b1-8bf0-622cb2ebb56f.png)

```yaml
aoiVersion: v1
kind: Pod
metadata:
	name: myapp-pod
	labels:
		app: myapp
		type: front-end
spec:
	containers:
	- name: nginx-container
		image: nginx
```

```yaml
kubectl apply -f pod.yaml -n dev
```

- 또는 Pod의 yaml로 정의해서 Namespace를 지정할 수 있다.
- 이는 리소스를 항상 동일한 Namespace에 생성되도록 보장하는 좋은 방법이다.

```yaml
aoiVersion: v1
kind: Pod
metadata:
	name: myapp-pod
	namespace: dev
	labels:
		app: myapp
		type: front-end
spec:
	containers:
	- name: nginx-container
		image: nginx
```

### Create Namespace

- apiVersion: v1
- kind: NameSpace
- metadata:
    - name: 네임스페이스 이름 지정

```yaml
kubectl create -f namespace-dev.yaml

# 커맨드로 만들기
kubectl create ns dev
```

### Namespace switch

![image](https://user-images.githubusercontent.com/76610641/219312090-a671d508-d157-4df3-8e9b-2938cc74c56c.png)

- kubectl get pods 커맨드를 입력하면, 리소스를 확인할 수 있다.
- dev namespace에 있는 리소스를 보려면 n 옵션을 사용해야 한다.
- 만약 dev namespace를 지정하지 않아도 되도록 namespace를 전환하고 싶다면 kubectl config 커맨드를 사용하면 된다.
    - 이 컨맨드를 실행하면 현재 컨텍스트의 Namespace를 dev로 설정한다.
    - 따라서 이후 kubectl get pods를 실행하면 namespace 옵션 없이도 dev 환경에 있는 파드를 조회할 수 있다.
- 모든 namespace에서 파드를 확인하고 싶다면 all namespace 옵션을 사용하면 된다.
    - kubectl get pods —all-namespaces

### Resource Quota

![image](https://user-images.githubusercontent.com/76610641/219312136-ea2f99c4-202d-4a95-bac9-e6b031a32a0e.png)

- Namespace에서 리소스를 제한하려면 리소스 할당량을 만들면 된다.
    - 할당량을 줄 namespace를 지정하고, 사양에 따라 10개의 파드, 10개의 CPU 유닛, 10GB 바이트의 메모리 등과 같이 할당량을 조정할 수 있다,

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
	name: compute-quota
	namespace: dev
spec:
	hard:
		pods: "10"
		requests.cpu: "4"
		requests.memory: 5Gi
		limits.cpu: "10"
		limits.memory: 10Gi
```

```yaml
kubectl create -f compute-quota.yaml
```

---

# Imperative vs Declarative

- 인프라 관리에 있어서 imperative(명령형) 접근 방식과 declarative(선언적) 접근 방식으로 분류된다.

### Imperative

- 목적지까지 도달하는 방법에 대해 step-by-step으로 말해야 한다.
- ;어떻게’를 중요하게 두는 방식이다.

### Declarative

- 최종 목적지만 지정하여 목적지까지 도달하면 된다.
- 단계별 지침을 말할 필요없이 최종 목적지만 지정하면 된다.
- ‘무엇을’할 것인지를 중요하게 두는 방식이다.

## Infrastructure as Code(IaC)

### Imperative 방식

- 코드형 인프라 세계에서 프로비저닝 인프라의 imperative 접근 방식의 예로는 단계별로 쓰여진 instructions 세트가 될 수 있다.
    - provisioning a VM named web server
    - installing the NGINX software on it
    - editing configuration file to use port 8080
    - setting the path to web files
    - downloading source code of repositories from Git
    - starting NGINX server

### Declarative

- 우리의 요구를 선언한다.
- 예를 들어, nginx 소프트웨어가 있는 웹 서버라는 이름으로 VM이 필요하고, 포트는 8080으로 설정하고 싶다면 단계별로 지침을 제공할 필요없이
- 시스템 또는 소프트웨어에 의해 수행된다.
- Ansible, Pupper, Terraform과 같은 오케스트레이션 툴이 범주에 속한다.
- 시스템이 충분히 지능적이여서 이미 벌어질 일을 알고 필요한 변경 사항만 적용한다.

## Kubernetes

- 쿠버네티스에서 오브젝트를 관리하는 **Imperative 방식**
    - 생성
        - 파드 생성시, kubectl run 커맨드를 사용하는 것과 같다.
        - Deployment 생성시, kubectl create deployment
        - Service 생성시, kubectl expose
    - 수정
        - kubectl edit
    - 확장
        - kubectl scale
    - 이미지 업데이트
        - kubectl set image
    - 파일 적용/수정/삭제
        - kubectl create -f
        - kubectl replace
        - kubectl delete
    - 오브젝트를 만들고, 업데이트하고, 삭제하는 니즈를 위해 어떻게 해야하는지 인프라에서 정확하게 말하고 있다.
- **declarative 방식**
    - 쿠버네티스 클러스터의 애플리케이션 및 서비스의 예상되는 상태를 정의하는 file 세트를 만든다.
    - 그리고 kubectl apply 커맨드로 쿠버네티스 configuration file을 읽을 수 있다.
    - 그리고 인프라를 예상되는 상태를 만들기 위해 해야할 일을 스스로 결정한다.
    - declarative 접근 방식은 오브젝트를 만들고, 업데이트하고, 삭제하기 위해 kubectl apply 커맨드를 실행한다.
    - **apply는 커맨드는 기존 구성을 확인하고, 시스템에 어떤 변경이 필요한지 파악한다.**

## Imperative Commands

- imperative(명령형) 커맨드를 사용하는 방법
    - 오브젝트 생성하기 위한 run, create, expose 커맨드와
    - 기존에 존재하는 오브젝트를 업데이트하기 위한 edit, scale, set 커맨드가 있다.
- 이러한 커맨드는 yaml을 다룰 필요가 없기 때문에 빠르게 오브젝트를 만들고 수정하는데 도움이 된다.
- 그러나, 기능이 제한적이고 multi-container 파드나 deployment를 생성하는 것과 같이 advanced use case의 경우 길고 복잡한 커맨드를 실행해야 한다.
- 또한 이러한 커맨드는 한 번 실행되고 잊혀진다.
- 해당 커맨드를 실행했던 사람의 세션 기록에서만 볼 수 있다.
- 따라서 다른 사람은 이 오브젝트가 어떻게 만들어졌는지 알기 어렵고 트래킹하고 어렵다.
- 따라서 크고 복잡한 환경에서는 이러한 커맨드로 작성하기 어렵다.
- 이 점이 오브젝트 configuration file로 오브젝트를 관리할 때 도움을 받을 수 있다.

## Imperative Object Configuration Files

- Definition file(=manifest file, configuration file)을 만드는 것은 우리가 원하는 것을 yaml 형식으로 정확히 기록하고 kubectl create 커맨드로 오브젝트를 만드는데 도움을 준다.
- yaml 파일이 있으면 git과 같은 코드 레포지토리에 저장할 수 있다.
    - 프로덕션에 반영되기 전에 변경사항을 검토하고 승인 프로세스를 넣을 수 있다.
- 향후 변경 사항이 있으면, 이미지 이름을 다른 버전으로 고치는 것과 같이 이를 처리하는데 여러 방법이 있다.

### **kubectl edit**

- 따라서 이 컨맨드가 실행되면 오브젝트를 만드는 것과 같이 yaml 파일이 열린다.
- 이것은 오브젝트를 만들 때 사용하는 파일이 아니고, 쿠버네티스 메모리에 있는 유사한  Pod 정의 파일이다.
- 우리는 해당 파일을 변경하고 저장하고 종료할 수 있다.
- 그러면 **변경 사항이 live object에 적용**된다.
- **kubectl edit 커맨드를 사용해 마든 변경사항은 실제로 어디에도 기록되지 않는다.**
- 변경이 적용되면 로컬 definition file만 남게 되는데, 이 definition file은 변경사항이 적용되지 않은 예전 버전이다.
- 향후 팀원이 kubectl edit 커맨드를 사용해 오브젝트가 변경된 사실을 모르고 오브젝트를 변경했다고 가정하자.
    - 변경사항이 적용될 때 이전 버전의 image가 손실된다.
    - 따라서 kubectl edit 커맨드는 더 이상 configuration file을 의존하지 않을 것이라 확실할 때 사용하는 것이 좋다.

### kubectl replace

- 로컬 버전 configuration file을 수정한 후, kubectl replace 커맨드를 실행하여 오브젝트를 업데이트 한다.
- 이런 방식은 변경사항이 기록되고, 변경 검토 프로세스의 일부를 추적할 수 있다.
- 때때로 오브젝트를 완전시 삭제하고 다시 만들어야 할 때가 있는데, 이 경우 force 옵션과 함께 동일한 커맨드를 실행한다.

지금까지 imperative 접근 방식이다. 어떻게 오브젝트를 생성하고 업데이트하는지 쿠버네티스에 지시하고 있기 때문이다.

- 오브젝트를 생성하기 위해 kubectl create 커맨드를
- 오브젝트를 수정하기 위해 kubectl replace 커맨드를
- 오브젝트를 삭제하기 위해 kubectl delete 커맨드를 실행한다.

그리고 이제 create 명령어를 입력하면 어떻게 될까?

- 오브젝트가 이미 존재하는데 create 커맨드를 실행하면 어떻게 될까? 파드가 이미 존재한다는 에러가 뜬다.
- 오브젝트를 업데이트할 때 replace 커맨드를 실행하기 전에 오브젝트가 먼저 존재하는지 확인해야 한다.
- 따라서 imperative 접근 방식은 항상 현재 구성을 알고 있어야 하고, 변경 전에 모든 것이 제자리에 있는지 체크해야 하므로 관리자에게 매우 부담이 된다.

## Declaraive

- 우리가 작업해 온 오브젝트 file을 동일하게 사용한다.
- 대신 create나 replace 커맨드가 아닌 kubectl apply 커맨드를 사용한다.
- **kubectl apply 커맨드는 아직 존재하지 않은 객체를 생성할 정도로 지능적이다.**
- 만약 configuration file이 여러개 있는 경우, 단일 파일이 아니라 폴더 단위로 지정하면 여러 오브젝트를 한 번에 생성할 수 있다.
- 오브젝트의 변경이 필요한 경우, 우리는 단순히 오브젝트의 configuration file을 업데이트하고, kubectl apply 커맨드를 실행해주면 된다.
- **오브젝트가 이미 존재한다는 것을 커맨드가 알고 있으며, 새로운 변경사항만 객체가 업데이트한다.**
    - 따라서 객체가 이미 존재한다는 에러나 업데이트를 적용할 수 없다는 에러는 발생하지 않는다.
- 이 방식은 항상 오브젝트를 업데이트하는 옳은 방법을 알아낼 것이다.
- 앞으로 애플리케이션의 어떤 변경사항이든, 우리는 단순히 로컬 디렉토리를 업데이트하고, kubectl apply 커맨드를 실행하는 것이다.

# Kubectl Apply Command

쿠버네티스 kubectl apply 커맨드가 어떻게 작동하는지 알아보자.

## kubectl apply

- apply 커맨드는 로컬 configuration file, 쿠버네티스의 live object definition, 마지막으로 적용된 구성을 변경사항을 결정하기 전에 고려한다.
- 따라서 apply 커맨드를 실행할 때, 오브젝트가 아직 존재하지 않은 경우, 오브젝트가 생성된다.
- 오브젝트가 생성되면 오브젝트 configuration은 쿠버네티스 내에서 생성된다.

- 오브젝트의 상태를 저장하는 추가 필드가 있다.
    - 쿠버네티스 클러스터에 있는 오브젝트의 live configuration이다.
    - 이것이 쿠버네티스가 어떤 접근 방식을 사용하여 오브젝트를 생성했던간, 내부적으로 정보를 저장하는 방식이다.
- 그러나 오브젝트를 생성하기 위해 kubectl apply 명령어를 사용하면 많은 작업을 수행한다.
- 우리가 작성한 로컬 오브젝트 configuration 파일의 yaml 버전은 json 형식으로 변환되고, 마지막으로 적용된 configuration으로 저장된다.
- 앞으로 오브젝트에 대한 모든 업데이트에 대해 어떤 변경이 live object에 적용되는지 비교해보자.
    - local file
    - last applied configuration
    - kubernetes(live objec configuration)
- 예시
    - 예를들어, nginx 이미지가 1.19로 업데이트되었을 때,
    - kubectl apply 커맨드는 **local 파일과 live configuration의 값을 비교**한다.
    - **차이가 있다면 live configuration이 새 값으로 업데이트** 된다.
    - 변경 후, 마지막으로 적용된 json 형식은 항상 최신 상태로 업데이트된다.

### **last applied configured**

- type label과 같은 필드가 삭제되었다고 가정하자.
- 이제 kubectl apply 명령어를 실행하면 last applied configuration에는 레이블이 있었으나, local configuration에는 없다는 것을 알 수 있다.
- 이것은 그 필드를 live configuration에서 제거함을 의미한다.
    - 따라서 live configuration 필드가 있고, 로컬이나 last에 존재하지 않는다면 필드는 그대로 남게 된다.
    - 그러나 local에는 필드가 누락되고 last에는 존재한다면 kubectl apply 명령어를 마지막으로 실행했을 때마다 해당 특정 필드가 있었고, 현재는 제거 중이라는 것을 의미한다.
- 따라서 last applied configuration은 로컬 파일에서 어떤 필드가 제거되었는지 파악하는데 도움이 된다.

### 세가지 파일의 위치

로컬 파일은 로컬 시스템에 저장되고, 라이브 오브젝트 configuration은 쿠버네티스 메모리에 있다.

last applied configuration이 저장되는 json file은 쿠버네티스 클러스터 자체의 live configuration에 last applied configuration라는 주석으로 저장된다.

따라서 적용 명령을 사용할 때 kubectl create 또는 replace와 같이 마지막으로 적용된 구성을 저장하지 말아야한다.

- 명령형 및 선언적 접근 방법을 혼합하면 안된다.
- 쿠버네티스는 적용된 명령을 사용하면 앞으로 변경이 있을 때마다 세 섹션을 모두 비교한다.
- 따라서 kubectl apply 명령을 사용해야만 완료되고, kubectl create, replace 명령은 last를 저장하지 않는다.
- kubectl apply 명령어를 사용할 때마다 live configuration 내 변경사항을 결정하기 위해 세가지 섹션을 모두 비교한다.
