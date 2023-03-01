# Secret

# 153. KubeConfig

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c81278dd-2adc-4875-8118-c70749453aaa/Untitled.png)

- my kube playground 클러스터가 kube-apiserver에 CA 인증서와 함께 curl을 보내면 API 서버에서 사용자 인증 유효성을 검사한다.
- kubectl을 사용하는 경우, 클라이언트 키, 인증서를 이용해 동일한 정보를 지정할 수 있다.

하지만 해당 정보를 매번 입력하기에는 지루하다..

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6b184e40-7b00-403e-9ab9-fa16032cfa7f/Untitled.png)

- 따라서 kubeconfig라는 구성 파일에 kubeconfig 옵셔으로 파일을 지정한다.
- 기본적으로 kubectl 도구는 config라는 파일을 찾는다.
    - HOME/.kube/config
- 따라서 파일 경로를 명시적으로 지정할 필요가 없다.

### KubeConfig File

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4856fe6a-82be-46f5-acdd-06e9c22f45b2/Untitled.png)

- config 파일은 세가지 섹션을 가지고 있다.
    - Clusters, Users, Contexts
    - Clusters : 접근하고자 하는 다양한 쿠버네티스 클러스터를 의미한다. 배포 환경에서 다양한 클러스터들을 가지고 있다. 그런것들이 들어가있다.
    - Users : 클러스터에 접근하고자 하는 사용자 계정을 의미한다.
    - Contexts : Cluster과 User를 엮어 놓은 것을 의미한다.
- 새로운 유저를 생성하는 것이 아니라, 기존에 있는 유저와 권한을 사용하고, 해당 유저가 어떤 클러스터를 사용할지 정의한다.
- 따라서, 모든 kubectl을 실행할 때, 사용자 인증서와 서버 주소를 각각 구체화하지 않아도 된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a1db686a-6fa3-4ac2-b97d-0e007ed52f3b/Untitled.png)

```yaml
apiVersion: v1
kind: Config

clusters:
- name: my-kube-playground
	cluster:
		certificate-authority: ca.crt
		server: https://my-kube-playground:6443
contexts:
- name: my-kube-admin@my-kube-playground
	context:
		cluster: my-kube-playground
		user: my-kube-admin
users:
- name: my-kube-admin
	user:
		client-certificate: admin.crt
		client-key: admin.key
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/36492cd8-b8b7-47c8-849d-e71e352ab5ab/Untitled.png)

- 파일이 준비되면, 해당 오브젝트를 create할 필요없다.
- 해당 파일은 kubectl로 인해 읽어질 것이다.

**kubectl은 어떤 context를 선택할까?**

- default context를 kubeconfig file에 current-context 필드를 통해 지정할 수 있다.

### kubeconfig view

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/20aea4b4-a930-498e-a458-08ac4e3af97e/Untitled.png)

```yaml
kubectl config view
```

- kubecongfig 파일을 지정하지 않으면, 홈 디렉터리 아래 .kube 폴더에 위치한 파일을 사용한다.

```yaml
kubectl config view --kubeconfig=my-custom-conf
```

- 해당 옵션을 지정해 config 파일을 지정해서 볼 수 있다.

### update current-context

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fc482e0c-83bb-4064-95d1-7e54a17c0120/Untitled.png)

```yaml
kubectl config use-context prod-user@production
```

### Namespaces

컨텍스트를 특정 네임스페이스로 바꾸기 위해 구성할 수 있을까?

- 컨텍스트는 namespace라는 추가적인 필드가 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/915d8af0-ef20-4d20-ae6d-7bd04d778273/Untitled.png)

### Certificates in KubeConfig

- certificates 부분에는 certificate 파일 경로를 지정할 수 있다.
- 또는 certificate-authority-data 필드를 통해 인증서를 인증할 수 있다.
- 해당 필드에 ca.crt를 base64 인코딩한 것을 넣어도 된다.

```yaml
cat ca.crt | base64
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0091390d-ca43-4815-acf0-f1f4eaee26f0/Untitled.png)

---

# 157. API Groups

- 마스터 노드 주소 + 6443(포트) + API Version으로 API 서버에 액세스할 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9592c27e-8756-40ad-9c5c-e65a382a51cb/Untitled.png)

- 파드의 목록을 얻으려면 curl api/v1/pods에 액세스한다.
- 쿠버네티스 API는 목적에 따라 여러 그룹으로 나뉜다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7eefed95-1b29-4756-ab56-bc6fc435aa99/Untitled.png)

### API & APIs

- API는 코어 그룹과 네임드 그룹으로 분류된다.
- 코어 그룹에는 핵심 기능이 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ef995f99-c871-4ca6-9002-4a5756a9e9b6/Untitled.png)

- 네임드 그룹에는 API가 더욱 체계화 되어 있다. 네임드 그룹을 통해 최신 기능을 사용할 수 있다.
    - apps, extensions, networking, storage, authentication, authorization 등이 있다.
    - /apps 안에는 deployemnts, replica sets, stateful sets
    - /networking.k8s.io 안에는 networkpolicy
    - /certificates.k8s.io 안에는 인증서 서명 요청

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bcc670bb-f9eb-471c-8094-43031cddd7ec/Untitled.png)

- 상단에 있는 것은 API 그룹이고 하단에 있는 것은 해당 그룹의 리소스이다.
- 또 해당 리소스에 대한 작업들이 나열되어 있다.

- curl [http://localhost:6443](http://localhost:6443) -k 를 나열하면 사용 가능한 API 그룹이 나열된다.
- 그리고 네임드 API 그룹 내에서 지원되는 모든 리소스 그룹을 반환한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3dd8d0f0-9cac-4bd2-a621-f781d9fac051/Untitled.png)

---

# 158. Authorization

### Why Authorization

- 관리자는 클러스터 내에서 모든 작업을 수행할 수 있다.
- 하지만 관리자는 목적에 따라 나뉘어져 있고, 액세스하는 오브젝트의 종류도 다르다.
- 그렇기에 모두가 같은 액세스 권한을 갖는 것은 위험이 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5f0fd016-227f-47bd-b6a0-7b2093b3d172/Untitled.png)

### Authorization Mechanisms

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4a65b4ec-35fd-4e5f-a68e-7938070cdb4a/Untitled.png)

### Node Authorization

- 클러스터 내부 액세스
- Kube API는 클러스터 내 노드의 kubelet에 액세스한다.
- kubelet은 API 서버에 액세스하여 서비스 및 포인트, 노드 및 파드에 대한 정보를 읽는다.
- kubelet은 상태와 같은 노드에 대한 정보와 함께 Kube API 서버에 보고한다.
- 이러한 요청을 node authorization이라는 특수 authorizer가 처리한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1fdd858d-c497-4cb8-ab2d-1986d774d807/Untitled.png)

### ABAC

- 외부 액세스에 관련한 속성 기반 권한이다.
- 사용자 또는 사용자 그룹을 권한 집합과 연결하는 곳이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bd514bb7-6ba2-49d9-961a-9fe17e6edc43/Untitled.png)

### RBAC

- 역할 기반 액세스 제어
- 사용자 또는 사용자 그룹을 일련의 권한과 직접 연결하는 대신 사용자를 위한 역할을 정의한다.
- 개발자에게 필요한 권한 세트로 역할을 만든 후 모든 개발자에게 해당 역할 연결한다.
- 사용자 액세스를 변경할 때마다 역할을 수정하기만 하면 되기 때문에 모든 개발자에게 즉시 변경이 용이하다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c8b5bf5d-26fc-4cf2-85f8-cf5e7c0f991d/Untitled.png)

### Webhook

- 외부에서 권한을 관리하는 경우 사용한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0b08144e-17ad-4ef2-9295-64566f0497ef/Untitled.png)

### Authorization Modes

- 추가적인 모드인 Always Allow, Always Deny
- 항상 허용 - 인증 확인을 수행하지 않고 모든 요청 허용
- 항상 거부 - 모든 요청 거부

디폴트는 항상 허용이다.

Kube API 서버에서 authorization 옵션 모드를 설정하면 된다.

여러 모드를 구성한 경우 지정한 순서대로 각 모드를 사용하여 요청에 권한이 부여된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/98cdc6dd-cf45-49da-8472-306040d08a65/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/21da2e56-2b9c-4604-bab2-fa57a9f19f39/Untitled.png)

- 모듈이 요청을 거부할 때마다 체인의 다음 모듈로 이동하고, 모듈이 요청을 승인하는 즉시 더 이상 확인하지 않고, 사용자에게 권한이 부여된다.

---

# 159. Role Based Access Controls

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e05dd99b-b120-42d4-a1b5-e25e19b9479e/Untitled.png)

---

# 162. Cluster Roles and Role Bindings
