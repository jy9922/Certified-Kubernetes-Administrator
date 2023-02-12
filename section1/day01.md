# ⚓ Day01 - Core Concepts

<br>

## Cluster Architecture

- 쿠버네티스의 목적은 애플리케이션을 호스팅하는 것이다.
- 쿠버네티스는 많은 애플리케이션 인스턴스를 자동화된 방식과 컨테이너 형태로 쉽게 배포할 수 있도록 하는 것이다.
    - 애플리케이션 내의 서로 다른 서비스간의 통신을 필요에 따라 쉽게 가능하게 할 수 있다.

### 예시 ⚓

- 쿠버네티스 클러스터는 노드로 구성되어 있다. 물리적 또는 가상, 온프레미스 또는 클라우드일 수 있다. 그리고, 컨테이너 형태로 애플리케이션을 호스팅한다.

![image](https://user-images.githubusercontent.com/76610641/218308541-836d4b20-db76-448b-a084-7db994c60ece.png)

- 위 그림에 있는 두 개의 선박이 컨테이너를 적재하고 있는 worker 노드이다.
    - worker 노드는 컨테이너를 적재할 수 있다.

❓ **그렇다면, 무엇이 컨테이너를 로드할까?**

💡 **Master 노드가 담당한다.**

![image](https://user-images.githubusercontent.com/76610641/218308584-83003a18-592e-48e3-82c6-dd40e29ab2b7.png)

- Master 노드는 다음과 같은 일을 한다.
    - 쿠버네티스 클러스터 관리를 위해 다른 노드에 관한 정보를 저장한다.
    - 어떤 컨테이너가 어떤 노드로 갈지 계획한다.
    - 노드와 컨테이너 등을 모니터링한다.
- Master 노드는 컴포넌트 동작을 제어하는 구성요소의 집합을 사용하여 위 작업을 수행한다.

### 구성요소

![image](https://user-images.githubusercontent.com/76610641/218308611-d39b6d24-c766-42ee-82b0-5aa4ebe608e9.png)

- 🚢 **Master Node 구성 요소**
    
    ![image](https://user-images.githubusercontent.com/76610641/218308627-248672f8-642d-4ab0-9eb7-f3d046a7be14.png)

    - **ETCD**
        - 고가용성을 보장하는 클러스터 형태의 저장소이다.
        - 정보를 키 값 형식으로 저장하는 데이터베이스
    - **Kube-scheduler**
        - 컨테이너를 배치할 올바른 노드를 식별한다.
        - 컨테이너 리소스 요구 사항, worker 노드 용량 또는 taint와 같은 정책이나 제약 조건 등을 참고하여 적절한 노드를 선택한다.
    - **Controller-Manager**
        - 다양한 영역을 관리하는 컨트롤러
        - 노드 컨트롤러 - 새 노드를 클러스터에 온보딩, 노드를  사용할 수 없게되는 상황 처리
        - 레플리케이션 컨트롤러  - 원하는 레플리카의 수의 컨테이너가 실행 중인지
    - **Kubernetes API**
        - 클러스터 내의 모든 작업을 오케스트레이션한다.
        - 외부 사용자가 사용하는 클러스터 관리 작업을 수행한다.
        - Worker 노드와 통신한다. ( kubelet과 통신)
- 🚢 **Worker Node 구성요소**
    
    ![image](https://user-images.githubusercontent.com/76610641/218308664-5e401412-8555-4bcb-910b-0f20fe801afa.png)
    - **Container Runtime**
        - 애플리케이션은 모두 컨테이너로 호스팅된다.
        - 따라서, 클러스터의 모든 노드는 컨테이너를 실행할 수 있는 소프트웨어가 필요하다.
        - ex. **ContainerD 혹은 Docker**
    - **kubelet**
        - 클러스터의 각 노드에서 실행되는 에이전트이다.
        - Kube API의 명령을 수신한다.
        - 필요에 따라 노드에 컨테이너를 배포하거나 제거한다.
        - Kube API에게 주기적으로 컨테이너 상태를 보낸다.
    - k**ube-proxy**
        - 클러스터 내 서비스 간 통신을 가능하게 해준다.

---

## ETCD

### **분산 키 - 값 저장소**

- 키 - 값 저장소란?
    
    ![image](https://user-images.githubusercontent.com/76610641/218308689-7447d646-f8fe-4fa9-83be-c10a1811b0c0.png)

    - 문서 또는 페이지 형태로 정보를 저장한다.
    - 각 개인은 문서를 얻고, 개인에 대한 정보는 파일 형태로 저장된다.
    - 따라서 한 파일을 변경해도 다른 파일에는 영향을 미치지 않는다.
    - 데이터가 복잡해지면서 JSON 또는 YANO와 같은 형태로 데이터를 저장한다.

### **ETCD 역할**

- 클러스터에 관한 정보를 저장한다.
    - Nodes, PODs, Configs, Secrets, Accounts, Roles, Bindings 등 에 관한 정보
- kubectl get 명령어는 ETCD 서버에서 온 정보들이다.
- 클러스터 내 모든 변경 사항 또한 etcd 서버에 업데이트 된다.
    - 추가 노드, 파드 배포 또는 레플리카 세트 등
- **ETCD 클러스터 배포**
    - 설치를 통한 배포와 kubeadm tool을 사용해서 배포하는 방법이 있다.
    - **바이너리 파일 설치를 통한 배포**
        - Master 노드에서 직접 바이너리 설치 및 etcd를 구성하는 것이다.
        - 서비스로 전달되는 옵션 1 : 인증서
        - 서비스로 전달되는 옵션 2 : 클러스터를 구성하기 위해서 주목해야하는 advertise-client-urls ( ETCD가 청취하는 주소, 서버IP와 포트 2379)
            - kube API 서버에서 etcd에 도달하려고 할 때 사용된다.
    - **kubeadm을 이용한 배포**
        - Pod로 etcd 서버를 배포한다.
        - Pod 내에서 etcd 유틸리티를 사용한다.
        
        ```bash
        # Pod로 etcd 서버로 배포된 것 확인
        kubectl get pods -n kube-system
        etcd-master
        
        # kubernetes에 저장된 모든 키 나열
        kubectl exec etcd-master -n kube-system etcdctl get / --prefix -keys-only
        # 쿠버네티스는 특정 디렉토리 구조에 데이터를 저장한다.
        # 루트 디렉토리는 레지스트리이며 그 아래 다양한 구성이 있다.
        ```
        
    - kubernetes는 고가용성을 위해 여러개의 Master Node가 있는데, ETCD 또한 Master 노드에 분산되어 여러개를 갖는다.
    - 따라서 분산되어 있는 ETCD는 서로에 대해 알고있어야 한다.
        
        ![image](https://user-images.githubusercontent.com/76610641/218308702-27a9b0c8-92f1-4c0c-a8f4-7defd6064c14.png)

        - 초기 클러스터 옵션을 통해 etcd 서비스의 다른 인스턴스를 저장해야 한다.

### **ETCD 설치 및 작동**

```bash
# 바이너리 다운로드 - GitHub 릴리즈에서 운영체제에 맞는 바이너리 다운로드
curl -L https://github.com/etcd-io/etcd/releases/download/v3.3.11/etcdv3.3.11-linux-amd64.tar.gz -o etcd-v3.3.11-linux-amd64.tar.gz

# 압축 풀기
tar xzvf etcd-v3.3.11-linux-amd64.tar.gz

# ETCD 서비스 실행
./etcd
```

- **ETCD 실행**
    - 기본 **2379 포트**에서 수신한다.
    - 정보를 저장하고 검색하기 위해 모든 클라이언트를 ETCD 서비스에 연결할 수 있다.
    - 기본 클라이언트는 ETCD 제어 클라이언트이다.
        - ETCD Command Line 클라이언트이다.
        - 이를 이용해 키-값 쌍을 저장하고 검색할 수 있다.

```bash
# ETCD 실행
./etcd

# 키-값 저장
./etcdctl set key1 value1

# 저장된 데이터 검색
./etcdctl get key1

# 옵션 검색
./etcdctl

# 버전 검색
./etcdctl --version
```

### **ETCD 버전**

- Version 0.1
    - 2013년 8월 출시
- Version 2.0 (공식 안정 버전)
    - 2015년 2월 출시
    - RAFT 합의 알고리즘 재설계, 초당 10,000회 이상의 쓰기 지원
- Version 3.0
    - 2017년 1월 출시
    - 최적화와 성능 향상 제공
- ETCD 프로젝트 → CNCF
    - 2018년 11월
- v2와 v3 사이에 많은 변화가 있었다.
- 따라서 API 버전도 v2에서 v3으로 변경되었다.
    - etcdctl 명령도 변경되었다는 것!
- 따라서 v2와 v3를 동시에 사용한다.
- 최신 버전에서 ETCD의 기본 API 버전은 v3으로 설정되어 있다.

```bash
# API v3로 변경하기

# API v3에서 작동 (환경변수 이용)
ETCDCTL_API=3 ./etcdctl version

# 환경변수로 내보내서 설정하기
export ETCDCTL_API=3 ./etcdctl version

# ETCDCTL v3
export ECTDCTL_API=3 ./etcdctl version
./etcdctl put key1 value1 # 값 설정 (set -> put)
./etcdctl get key1 # 값 가져옴
```

⚙️ ETCDCTL

- ETCD와 상호작용하는데 사용하는 CLI 도구이다.
- ETCDCTL은 2개의 API 버전 (v2, v3)을 사용한다.
- 기본적으로 v2를 사용하도록 설정되어 있다.**

- ————v2————
- etcdctl backup
- etcdctl cluster-health
- etcdctl mk
- etcdctl mkdir
- etcdctl set

- ————v3————
- etcdctl snapshot save
- etcdctl endpoint health
- etcdctl get
- etcdctl put

**환경변수를 이용하여 ECTDCTL_API를 v3로 바꾸는 방법**

export ETCDCTL_API=3


---

## Kube-api server

![image](https://user-images.githubusercontent.com/76610641/218308814-6caf11fd-adde-4692-b931-76255d4a2955.png)
- 쿠버네티스 기본 관리 구성 요소이다.
- kubectl 명령을 입력하면 kube-apiserver에 요청을 인증하고 검증한다.
- 그후 데이터를 etcd 클러스터에서 검색하고, 응답한다.
- kubectl 명령을 사용하지 않고 API를 직접 호출할 수도 있다.
    
    curl -X POST /api/v1/namespaces/default/pods
    

### create pod 동작 과정

- User가 명령어를 입력한다.
- 요청에 대한 인증, 인가를 통해 유효성을 검사한다.
- 이후 API 서버는 노드에 할당하지 않고, Pod 객체를 생성한다.
- ETCD 서버에 해당 정보를 업데이트 한다. (Pod를 생성한 User를 업데이트한다)
- 스케줄러가 API 서버를 지속적으로 모니터링한다. → 할당된 노드가 없는 새로운 Pod가 생성된 것을 확인한다. 이후 스케줄러는 새 Pod를 위치시킬 올바른 노드를 식별한다. 이후 이를 kube-apiserver에 전달한다.
- kube-apiserver은 이에 대한 정보를 ETCD에 업데이트한다.
- 이후 kube-apiserver은 적절한 노드의 kubelet에 해당 정보를 전달한다.
- kubelet은 해당 노드에 Pod를 생성하고, 컨테이너 런타임 엔진에 애플리케이션 이미지 배포를 지시한다.
- 해당 작업이 완료되면 kubelet이 상태를 업데이트하고, 이를 kube-apiserver에 전달한다.
- kube-apiserver은 해당 내용을 ETCD에 업데이트한다.

### kube-apiserver의 역할

![image](https://user-images.githubusercontent.com/76610641/218308835-7efb0338-9368-4925-8626-1f06c709ed74.png)

- 인증, 인가
- 요청 확인
- ETCD 데이터 저장소에 데이터 검색 및 업데이트
    - ETCD 데이터 저장소와 직접 상호작용하는 유일한 컴포넌트이다.
- kubelet, kube-controller-manager, scheduler 업데이트 수행

### kube-apiserver with 바이너리

Master 노드에서 서비스로 실행되도록 구성한다.

```bash
# kubernetes 릴리스 페이지의 바이너리 다운로드
wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-apiserver
```

**클러스터의 다양한 구성요소 (인증서)**

![image](https://user-images.githubusercontent.com/76610641/218308848-2257e98e-5e24-4fa5-8382-cf8a410ca77d.png)

- 서로 통신하기 위해 다양한 인증 방식이 있다. (승인, 암호화 및 보안)
- 인증서가 가장 중요한 부분!

**ETCD 서버 위치 지정**

![image](https://user-images.githubusercontent.com/76610641/218308868-29bc35e4-cc2a-4c71-8aa2-dd10cd45dfa7.png)

- ETCD 서버 위치를 지정한다.
- ETCD와 kube-apiserver와 연결하는 방법이다.

**kubeadm으로 설정하여 kube-apiserver 옵션 검사**

![image](https://user-images.githubusercontent.com/76610641/218308881-840e86f4-8345-477c-9365-ae204e3acf03.png)

- kubeadm은 kubeadm-apiserver을 마스터 노드의 kube-system 네임스페이스 Pod로 배포한다.
- 따라서 Pod 정의 파일 내에서 옵션을 볼 수 있다.

![image](https://user-images.githubusercontent.com/76610641/218308896-387fba43-266a-44cd-825e-f14a010467b1.png)

**kubeadm가 아닌 설정에서 옵션 검사**

![image](https://user-images.githubusercontent.com/76610641/218308909-05f9b626-2a04-47eb-accb-d2f0e462d601.png)

**실행중인 프로세스 나열을 통해 kube-apiserver 옵션 검사**

![image](https://user-images.githubusercontent.com/76610641/218308920-6dafa23c-252c-4bfb-8e36-78d8c6d77487.png)

---

## Kube-Controller-Manager

- kubernetes에서 다양한 컨트롤러 관리한다.
- 컨트롤러는 지속적으로 시스템 내의 다양한 구성요소 상태를 모니터링한다.
- 전체 시스템을 원하는 상태로 가져오기 위해 노력한다.
- Node Controller, Replication Controller 등 다양한 컨트롤러가 있다.
    
    ![image](https://user-images.githubusercontent.com/76610641/218308936-6f3af904-6b02-4b4f-a8d9-32db67224633.png)

### **Node Controller**

![image](https://user-images.githubusercontent.com/76610641/218308968-151199b5-053b-4778-b3c2-63a66ca6a256.png)

- 노드의 상태를 모니터링하고, 애플리케이션 실행을 유지하기 위해 필요한 기능을 수행할 수 있도록 한다.
- kube-apiserver을 통해 수행한다.
- 노드의 상태를 5초마다 모니터링하고, 만약 노드가 멈췄다면 40초 뒤 노드를 unreachable로 표시한다.
- 이후, 5분 동안 다시 되돌아 올 수 있게 기다린다.
- 만약 그 이후에도 안 돌아온다면 해당 노드에 할당된 Pod를 제거하고, 다른 건강한 노드를 골라 replica 수에 맞춰 재할당해준다.

### **Replication Controller**

- replica set 상태를 모니터링하고, 요구하는 Pod 수를 보장할 수 있도록 한다.
- Pod가 죽으면 다른 Pod를 생성한다.

### Controller Manager 설치

- 여러가지 컨트롤러는 Kubernetes Controller Manager 단일 프로세스로 패키지된다.
- 따라서, Kubernetes Controller Manager를 설치하면 다른 컨트롤러도 모두 설치된다.

```bash
# kubernetes 릴리스 페이지에서 다운로드
wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-controller-manager
```

**설치 시 옵션 목록 확인**

![image](https://user-images.githubusercontent.com/76610641/218308977-3fc93b27-0a5f-465d-95c2-8146397b2206.png)

**kubeadm tool로 설정하는 경우 옵션 보기**

![image](https://user-images.githubusercontent.com/76610641/218308992-70a31a63-bf3b-41ae-9af9-9e766dc7ca45.png)

- kube-system 네임스페이스에 Pod로 배포된다.
- Pod 정의 파일내 /etc/kubernetes/manifests/에 정의된 폴더에서 확인할 수 있다.

![image](https://user-images.githubusercontent.com/76610641/218309010-ff79a56d-a598-4345-89a0-61342123ddb4.png)

**kubeadm tool로 설정하지 않은 경우 옵션 보기**

![image](https://user-images.githubusercontent.com/76610641/218309037-5cd24725-ca77-4a3e-9bee-0d134fb2749a.png)

![image](https://user-images.githubusercontent.com/76610641/218309024-f3b8b302-9e40-4088-96f4-3a7ca50ff088.png)

---

## Kube Scheduler

- 노드에서 Pod 예약을 담당한다.
- 실제로 배치는 담당하지 않는다. (실제 Pod는 Worker 노드의  Kubelet 만든다)
- 즉, 어떤 Pod가 어디로 가는지 결정한다.

### 스케줄러 수행 방법

- 스케줄러는 각 Pod를 확인한다.
- 그리고 리소스 등을 확인하여 최적의 노드를 찾으려고 한다.
- Pod에 가장 적합한 노드를 식별한다.
    1. Pod의 리소스에 맞지 않은 노드를 필터링한다.
    2. 가능한 Pod를 중 노드의 순위를 매긴다.
        1. 우선순위 기능을 사용하여 점수를 할당한다.
    - 스케줄링 방법은 커스터마이징이 가능하다.

### 스케줄러 설치

```bash
# 릴리스 페이지에서 다운로드
wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-scheduler
```

**kubeadm 도구 설치시 scheduler 옵션 보기**

- Master 노드의 kube-system 네임스페이스에서 Pod로 배포되기 때문에, Pod 정의 파일 내에서 옵션을 볼 수 있다.

![image](https://user-images.githubusercontent.com/76610641/218309048-dd334827-803a-46f2-87de-0d709605e787.png)

**실행중인 프로세스로 옵션 보기**

![image](https://user-images.githubusercontent.com/76610641/218309061-d9b486aa-ee18-4192-8b93-fe90f20e4e5c.png)
