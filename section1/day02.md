# Kubelet

Kubelet은 배의 선장과 같다.

![image](https://user-images.githubusercontent.com/76610641/218788755-a436751b-3ba3-4aea-9f9c-705d90cde8c2.png)

- Kubelet은 마스터 노드와 유일한 접점이다.
- 마스터의 스케줄러가 지시한대로 Pod를 생성하거나, 컨테이너 혹은 노드의 상태를 정기적으로 보고한다.
- Kubernetes 워커 노드에 있는 Kubelet은 Kubernetes 클러스터에 노드를 등록한다.
- Kubelet은 노드의 Pod나 컨테이너를 로드하라는 지시를 받았을 때, Docker와 같은 컨테이너 엔진에게 이미지 pull을 요청하고 인스턴스를 실행한다.
- 이후 Kubelet은 Pod와 컨테이너의 상태를 지속적으로 모니터링하고, kubeAPI 서버에 필요시 보고한다.

### install kubelet

- kubeadm을 이용하여 클러스터를 배포하는 경우
    - 다른 컴포넌트와 다르게 **kubelet을 자동으로 배포되지 않는다**.
    - kubelet은 워커 노드에 수동으로 설치해야 한다.
    - 설치 프로그램을 다운로드하고, 압축을 풀고, 서비스를 실행한다.
    ![image](https://user-images.githubusercontent.com/76610641/218789070-06eff36f-0172-49d3-bcbb-dd28b31b3b85.png)

### Kubelet Option
- 실행 중인 Kubelet 프로세스와 적용된 옵션을 볼 수 있다.
    
    ```bash
    ps -aux | grep kubelet
    ```
    
    ![image](https://user-images.githubusercontent.com/76610641/218789359-095b632b-a519-4751-bfe7-a1d9f4cbb853.png)

---

# Kube Proxy

- Kubernetes 클러스터 내에서 모든 파드는 다른 파드와 소통할 수 있다.
- 이는 클러스터에 대한 파드 네트워킹 솔루션 배포를 통해 수행된다.
- 파드 네트워크는 클러스터의 모든 노드에 걸쳐져 있고, 모든 파드에 연결되어 있는 하나의 inter virtual network이다.
- 이 네트워크를 통해 파드들은 서로 소통할 수 있다.
- 이런 네트워크를 배포하기 위한 많은 솔루션들이 있다.

### ⚓ 예시

![image](https://user-images.githubusercontent.com/76610641/218791533-d838c4d9-1782-4a5b-b572-b42bbf36b3e9.png)

- 첫번째 노드에 웹 애플리케이션이 하나 있고, 두번째 노드에는 데이터베이스 애플리케이션이 있다.
- 웹 앱은 파드 IP를 사용하여 간단하게 데이터베이스에 접근할 수 있다.
- 그러나 데이터베이스 파드 IP가 항상 동일하다는 보장은 되지 않는다.
- 웹 애플리케이션이 데이터베이스에 접근하는 더 나은 방법은 서비스를 사용하는 것이다.
- 따라서 클러스터 전체에 데이터베이스 애플리케이션을 노출하는 서비스를 만든다.
- 이제 웹 애플리케이션은 서비스 이름인 DB를 사용하여 데이터베이스에 접근할 수 있다.
- **서비스는 IP주소도 할당받기 때문에** 파드가 서비스 IP나 서비스 이름을 사용하여 서비스에 접근할 때마다, 서비스는 트래픽을 데이터베이스 파드로 전달한다.

### 서비스란 무엇이고, 어떻게 IP에 도달하는가?

- 서비스는 실물이 아니기 때문에 파드 네트워크에 조인할 수 없다.
- 서비스는 파드와 같은 컨테이너가 아니기 때문에 어떠한 인터페이스도, listening 상태의 프로세스도 아니다.
- **쿠버네티스 메모리에 살아있는 가상 컴포넌트**이다.

그렇다면, 클러스터의 모든 노드에서 어떻게 접근이 가능하게 되는것일까?

- 서비스는 클러스터의 모든 노드에 접근이 가능해야 한다.
- 여기서 **kube-proxy가 사용**된다.

### kube-proxy란?

- kube-proxy란 쿠버네티스 클러스터의 각 노드에서 실행되는 프로세스이다.
- kube-proxy의 임무는 **새로운 서비스를 찾고, 새로운 서비스가 생성될 때마다 각 서비스에 대한 트래픽을 백엔드 파드로 전달하기 위한 적절한 규칙을 생성**하는 것이다.ㅇ
- 규칙을 생성하는 방법에는 iptables 규칙을 사용하는 것이다.ㅇ
- 지금과 같은 케이스에서 kube-proxy는 클러스터의 각 노드에서 서비스 IP로 향하는 트래픽을 전달하기 위해 **iptables 규칙을 생성**한다.
- 실제 파드 IP인 10.32.0.15를 서비스 IP 10.96.0.12로 변경한다. 이것이 kube-proxy가 서비스를 구성하는 방법이다.

### install kube-proxy

- 쿠버네티스 릴리스 페이지에서 kube-proxy 바이너리 파일을 다운받고 압축을 풀고, 서비스로 실행한다.

![image](https://user-images.githubusercontent.com/76610641/218789496-27eb56c3-177c-4e01-86f3-975e969e5338.png)


- kubeadm 도구는 kube-proxy를 각 노드에 DaemonSet 형태로 배포한다. 따라서 클러스터 각 노드에서 단일 파드가 항상 배포된다.

![image](https://user-images.githubusercontent.com/76610641/218789565-f1c7186c-1f19-4710-ab9c-d0d8f67ae9d2.png)

---

# Pod

파드에 대한 이야기를 하기 전, 가정을 몇 가지 해보자.

- 애플리케이션이 이미 개발되어 도커 이미지에 build되어서 도커 허브와 같은 도커 레포지토리에서 사용 가능하다.
    - 따라서 쿠버네티스는 도커 이미지를 pull down할 수 있다.
- 쿠버네티스 클러스터가 이미 셋업되어 있으며 작동중이다.
- 모든 서비스는 실행 중인 상태이다.

### Pod란

- 쿠버네티스의 궁극적인 목표는 클러스터에서 워커 노드로 구성된 컨테이너 형태로 애플리케이션을 배포하는 것이다.
- 그러나, 쿠버네티스는 컨테이너를 워커 노드에 직접 배포하지 않는다.
- **컨테이너들은 Pod라는 쿠버네티스 오브젝트로 캡슐화되어 있다.**
- **파드는 애플리케이션의 단일 인스턴스이며, 쿠버네티스에서 만들 수 있는 가장 작은 오브젝트**이다.

![image](https://user-images.githubusercontent.com/76610641/218789645-99fe4454-3913-4842-8f27-1cd8546d4622.png)

- 쿠버네티스 클러스터에 하나의 노드가 있고, 노드 안에서 애플리케이션 인스턴스 하나가 컨테이너로 캡슐화된 파드에서 실행되고 있다고 가정해보자.

![image](https://user-images.githubusercontent.com/76610641/218789727-625e4f35-93ff-4174-a33f-9c3395804815.png)

❓ **애플리케이션에 접속하는 유저수가 증가해서, 애플리케이션을 확장해야 한다면 어떻게 해야할까?**

![image](https://user-images.githubusercontent.com/76610641/218789785-abecdde2-b03a-4f2d-b0d7-1b7cf25879a4.png)

- 로드를 분산하기 위해 **애플리케이션 인스턴스를 추가**해야한다.

❓ **그렇다면 인스턴스는 어디에 추가해야할까?**

- 같은 Pod 내에 새 컨테이너 인스턴스를 추가해야할까? 아니다. **새 인스턴스와 함께 새로운 Pod를 만든다.**

![image](https://user-images.githubusercontent.com/76610641/218789860-153dab36-50e6-41bf-9e6b-b8196e4acbd6.png)

- 따라서 같은 노드 안에 두 개의 별도 파드에서 애플리케이션이 실행되는 두개의 인스턴스가 있게 된다.

❓ **유저수가 더 많아져서 노드에 충분한 공간이 없다면 어떻게 할까?**

- 새 노드를 만들어서 추가 파드를 배포하면 된다.
- 클러스터의 물리적 용량을 확장하기 위해 새로운 노드를 추가한다.

![image](https://user-images.githubusercontent.com/76610641/218789910-aa7f17e6-e677-4dec-86a5-82fd9dd300d0.png)

중요한 점은, **파드는 일반적으로 애플리케이션을 실행하는 컨테이너와 1:1 관계를 가진다는 것**이다.

- 확장하려면 새로운 Pod를 만들고, 축소하려면 기존 Pod를 삭제하면 된다.
- 기존 Pod에 컨테이너를 추가하는 것이 아니다!

### Multi-Container Pod

- 파드가 일반적으로 컨테이너와 1:1 관계를 가진다고 했다.
- 그렇다면 쿠버네티스 한 파드에는 하나의 컨테이너만 가질 수 있게 제한해둔 것일까? **No!**
- **한 파드는 같은 종류의 컨테이너가 아니라면, 여러 컨테이너를 가질 수 있다.**
- 우리의 의도가 애플리케이션을 확장하는 것이라면 같은 종류의 인스턴스(컨테이너)를 추가해야 하는 것이므로 추가 Pod를 생성해야 한다.
- 그러나 확장하는 것이 아닌 다른 시나리오가 있을 수 있다.
- **웹 애플리케이션의 경우, 유저 처리, 업로드된 파일 데이터 프로세싱 등 지원 작업을 수행하는 helper 컨테이너가 있을 수 있다.**

![image](https://user-images.githubusercontent.com/76610641/218789959-2fdd686a-4c54-472c-8dbc-4d6cdd87d75f.png)

- 이 경우 우리는 helper 컨테이너가 애플리케이션 컨테이너와 함께 있길 원할 것이다.
- 따라서 이런 케이스에는 한 파드 내에서 여러 컨테이너를 가질 수 있다.
- 이 경우, 두 컨테이너가 동일한 Pod 안에 있기 때문에 새 애플리케이션 컨테이너가 생성되면 헬퍼도 생성되고, 기존 애플리케이션 컨테이너가 죽으면 헬퍼도 죽는다.
- 두 컨테이너는 동일한 네트워크 space를 공유하기 때문에 **서로  localhost로 참조하여 직접 소통**할 수 있다. 또한 **동일한 저장 공간을 쉽게 공유**할 수도 있다.

**참고💡  ) 멀티 컨테이너 파드는 다양한 패턴을 가진다.**

- 사이드카 패턴, 어뎁터 패턴, 앰버서더 패턴이 있다.
    
    [https://seongjin.me/kubernetes-multi-container-pod-design-patterns/](https://seongjin.me/kubernetes-multi-container-pod-design-patterns/)
    

### Pod - Docker 예시

- 쿠버네티스를 제외하고 도커 컨테이너에 대한 이야기를 해보자.
- 현재 우리의 애플리케이션을 도커 호스트에 배포하기 위한 프로세스나 스크립트를 개발 중이라고 가정하자.
- docker run python-app 커맨드로 간단하게 애플리케이션을 배포할 수 있고, 애플리케이션이 제대로 실행되면 유저도 접근할 수 있게된다.
- 부하가 증가하면, docker run python-app 커맨드를 여러번 실행함으로써 애플리케이션의 더 많은 인스턴스를 배포한다.
- 이렇게 간단하게 동작하게 되며 우리는 해피하게 된다.

### 헬퍼 컨테이너의 등장

- 애플리케이션은 더 개발되고, 아키텍쳐의 변화를 겪고, 성장하고 복잡해지고 있다.
- **애플리케이션을 도와주는 헬퍼 컨테이너가 생기게 된다.**

![image](https://user-images.githubusercontent.com/76610641/218791691-837608f9-d4ab-4706-a507-7010e6b1287b.png)

- 이 헬퍼 컨테이너는 다른 곳에서부터 데이터를 가져오거나 데이터를 처리해준다.
- 이러한 **헬퍼 컨테이너는 애플리케이션 컨테이너와 일대일관계를 유지**한다.
- 또한, 헬퍼 컨테이너는 애플리케이션 컨테이너와 직접 소통고 데이터 접근이 필요하다.
- 이를 위해 우리는 어떤 앱이 어떤 헬퍼 컨테이너와 연결되어 있느지 매칭하는 맵을 관리하고 있어야 한다.
- 즉, 이러한 컨테이너 사이에서 네트워크 연결을 설정해야 한다.
- 이 네트워크 설정은 docker run helper -link app1과 같은 커맨드를 사용한다.
- 또한 데이터 접근을 위해 공유 가능한 볼륨을 생성하고, 컨테이너들에게 공유해야 한다. 그리고 이러한 것들도 맵으로 관리해야 한다.
- 가장 중요한 것은 애플리케이션 상태를 모니터링해야 한다.
    - 애플리케이션 컨테이너가 죽으면, 헬퍼 컨테이너도 더이상 필요하지 않기 때문에 수동으로 죽여야 한다.
    - 새로운 애플리케이션 컨테이너가 배포되면, 새로운 헬퍼 컨테이너도 배포해야 한다.

### Pod의 등장

![image](https://user-images.githubusercontent.com/76610641/218790093-441f9d7a-1710-4b48-ae3b-9fff795b4c34.png)

- **Pod를 사용하면 쿠버네티스가 이 모든 작업을 자동으로 수행**해준다.
- Pod가 어떤 컨테이너로 구성되어 있는지 정의하기만 하면 된다.
- 기본적으로 **Pod의 컨테이너는 동일한 스토리지, 동일한 네트워크 네임스페이스에 액세스 권한을 갖는다.**
- 또한 파드의 컨테이너들은 함께 생성되고 함께 파괴된다.
- 만약 우리의 애플리케이션이 복잡하지 않아서, 컨테이너 하나만 있어도 된다 하더라도, 파드는 생성해야 한다.
- 이렇게 **파드를 생성하는 것이 애플리케이션 아키텍쳐가 변경되거나 확장될 때를 대비해서 장기적으로 좋다.**
- 멀티 컨테이너 파드가 굉장히 드문 사례이기 때문에 앞으로는 파드당 컨테이너 한 개가 있다고 가정하고 설명하겠다.

### How to Deploy Pod

- kubectl run 커맨드는 파드를 생성하여 Docker 컨테이너를 배포하는 것이다.
- kubectl run nginx는 파드를 자동으로 생성하고, nginx Docker 이미지의 인스턴스를 배포한다.
- 이때 nginx 이미지를 가져오기 위해 파라미터를 지정해준다.

```bash
kubectl run nginx --imgae=nginx
```

- 도커 허브 레포지토리에서 nginx 이미지를 다운로드하게 된다.
    - 도커 허브는 다양한 애플리케이션들의 최신 도커 이미지들이 모여 있는 public 저장소이다.
    - 그리고, 우리는 도커 허브나 조직의 private 저장소에서 이미지를 가져올 수 있도록 쿠버네티스를 구성할 수 있다.

### 사용 가능한 Pod 보기

```bash
kubectl get pods
```

커맨드를 통해 클러스터의 Pod 목록을 볼 수 있다.

![image](https://user-images.githubusercontent.com/76610641/218790154-01fb0fcc-be3c-4454-bc15-63b1a0083b35.png)

- 상태가 ContainerCreating인 파드를 볼 수 있다.
- 이 파드는 곧 Running 상태로 바뀌고, 실제 실행된다.
- 현재 상태는 외부 사용자가 웹 서버에 접근할 수 있도록 만들지 않았다. 노드 내부에서만 접근 가능하다.

---

# Pod with YAML

- YAML 기반의 configuration file을 사용하여 파드를 만드는 방법에 대해 알아보자.
- 쿠버네티스는 파드, 레플리카, 배포, 서비스 등과 같은 객체를 생성하기 위해 YAML 파일을 입력으로 사용한다.
- YAML은 다음과 같은 구조를 가진다. Kubernetes definition file은 최상위 레벨에 항상 아래 4가지가 포함된다.
    - apiVersion
    - kind
    - metadata
    - spec
- 위 4가지는 최상위 레벨(루트 레벨) 속성들이며 필수로 입력해야하는 속성이다. 따라서 configuration file에 반드시 필요한 것들이다.

![image](https://user-images.githubusercontent.com/76610641/218790201-21787669-06d0-444f-8d23-1401d000b592.png)

### apiVersion

- 오브젝트를 생성하는데 사용하는 kubernetes API 버전을 말한다.
- 우리가 어떤 것을 만드려고 하느냐에 따라 올바른 apiVersion을 사용해야 한다.

![image](https://user-images.githubusercontent.com/76610641/218790271-2ab8c0cc-0703-4418-9ebd-5e70ee2530eb.png)

- Pod 작업을 하고 있기 때문에 v1으로 설정한다.

![image](https://user-images.githubusercontent.com/76610641/218790396-3dd5f24a-9d53-4b28-a30a-421a786ce0d3.png)

### kind

- 우리가 만드려고하는 오브젝트 타입을 말한다.

![image](https://user-images.githubusercontent.com/76610641/218790271-2ab8c0cc-0703-4418-9ebd-5e70ee2530eb.png)

![image](https://user-images.githubusercontent.com/76610641/218790451-77b0a388-0903-4e06-b76c-77a58f6291ea.png)

### metadata

- metadata는 name, labels 등과 같이 오브젝트에 대한 데이터들이다.
- apiVersion과 kind는 String 값을 넣어줬는데, metadata에는 dictionary 값이 들어간다.

![image](https://user-images.githubusercontent.com/76610641/218790497-0c45beda-0bf4-4c3e-9fbc-36130a71321f.png)

- metadata 아래에 있는 값은 오른쪽으로 들여쓰기가 되어 있어서 name과 label이 metadata의 하위항목이라는 것을 알 수 있다.
- name과 label 앞에 공백을 몇 칸 두는지는 중요하지 않지만(공백 2칸을 권장!)
- 둘은 sibling 항목이기 때문에 동일하게 공백을 주어야 한다.
- name에는 Pod의 이름을 지정할 String 값이 들어간다.
- labels에는 오브젝트를 식별하기 위한 값이며, dictionary 값이 들어간다.
    
    ![image](https://user-images.githubusercontent.com/76610641/218790558-46b22537-3352-4595-b272-f89c12d53fe3.png)
    
    - labels는 원하는대로 key와 value쌍을 넣을 수 있다.
    - 프론트엔드 애플리케이션에서 수행되는 수백 개의 Pod가 있고, 백엔드 애플리케이션이나 데이터베이스에서 실행되는 또다른 수백 개의 Pod가 있다고 가정한다.
    - 이러한 Pod들이 모두 배포된 두에 프론트엔드, 백엔드끼리 그룹화하는 것은 어려울 것이다.
    - 따라서 **Pod에 프론트엔드, 백엔드, 데이터베이스 등 label을 달아 놓으면 나중에 이 label을 보고 Pod를 필터링할 수 있다.**
- metadata 아래에는 name, labels 등 kubernetes에 미리 지정된 속성들만 추가할 수 있다. 따라서 마음대로 속성을 추가할 수 없다.
- metadata 아래의 labels 아래에는 우리 마음대로 키-값 쌍을 추가할 수 있다.

### spec

- 우리가 만들 오브젝트에 따라, 쿠버네티스에 제공할 오브젝트 관련 추가 정보를 작성하는 곳이다.
- 이 추가 정보들은 오브젝트마다 다르다.
- 공식 문서를 참조하여 각각에 적합한 포맷을 얻어야 한다.
- 지금은 컨테이너 1개를 가진 파드 하나만 생성해보자.

- spec은 dictionary 형태로 작성된다.
- spec 아래에 containers 속성을 가진다.
    - 파드는 여러 개의 컨테이너를 가질 수 있기 때문에 containers 속성은 리스트이다.
    
    ![image](https://user-images.githubusercontent.com/76610641/218790621-4910cdee-a5b0-4d90-bbff-035c631bd1c6.png)
    
    - containers 리스트의 아이템은 name과 image가 key 값인 dictionary가 들어간다.
    - name 앞에 - 가 붙었는데 이는 리스트의 첫번째 아이템이라는 뜻이다.
    - 컨테이너 이름은 nginx-contianer이고, image는 도커 레포에 있는 nginx 이미지이다.

![image](https://user-images.githubusercontent.com/76610641/218790662-deb5f73b-1b65-433b-b4cf-713a59f893f0.png)

파일 완성 후, **kubectl create -f pod-definition.yml** 커맨드를 실행하면 kubernetes는 파드를 생성한다.

### Pod 생성 with YAML 데모

```bash
vi pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
    tier: frontend
spec:
  containers:
  - name: nginx
    image: nginx
  - name: busybox
    image: busybox
```

```yaml
# wq로 저장 후 나가기

# Pod 생성
kubectl apply -f pod.yaml

# Pod 확인
kubectl get pods

# 파드 이름으로 정확하게 확인
kubectl describe pod nginx
```

---

### 문제풀이

헷갈린거 문제 풀이

Q12. Create a new pod with the name redis and with the image redis123.

```yaml
kubectl run redis --image=redis123 --dry-run=client o yaml > redis-pod.yaml
```

![image](https://user-images.githubusercontent.com/76610641/218790807-3ec850dc-2d3b-4bb9-b194-e95937b56d38.png)

Q13. Now change the image on this pod to redis. Once done, the pod should be in a running state.

```bash
vi redis-pod.yaml
# 이미지 부분 수정 후 저장

kubectl apply -f redis-pod.yaml

# Running 확인
kubectl get pods
```
![image](https://user-images.githubusercontent.com/76610641/218791188-8bb47710-653a-44fb-b201-f3d76daf49d7.png)
