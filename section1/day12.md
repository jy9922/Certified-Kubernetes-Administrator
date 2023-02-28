# Day02

# 144. Kubernetes TLS

- 개인키와 공개키의 안전한 연결을 위해서 서비스 인증서를 사용한다.
- 인증 기관은 CA가 있고, 서버 인증서에 서명하는데 사용한다. (루트 인증서)
- 세 가지 유형의 인증서가 있다.
    - 서버에 구성된 서버 인증서
    - CA 서버에 구성된 루트 인증서
    - 클라이언트에 구성된 클라이언트 인증서
- 공개키가 포함되어 있는 인증서 = crt
    - 서버 인증서 서버 server.pem 또는 server.crt
    - 클라이언트 인증서 clinet.crt 또는 client.pem
- 개인키 경우 = key라는 단어가 포함되어 있다.
    - 파일이름.key 또는 server-key.pem
- key라는 단어가 있으면 개인키, 없으면 공개키 또는 인증서다.

### Kubernetes Cluster

- 마스터 노드와 워커 노드 세트로 구성되어 있다.
- 노드 간의 통신은 보안되어야 하고, 암호화 되어야 한다.

## kubernetes 서버 구성요소

- kubectl 유틸리티와 kubernetes API를 직접 액세스하는 방법으로 사용자가 접근할 때 안전한 TLS 연결을 설정해야 한다.
- 모든 구성요소 간 통신에서 kubernetes 클러스터 보안이 필요하다.
- 클러스터 내에 다양한 서비스가 가져야 하는 두 가지 주요 요구 사항이 있다.
    - 서버 인증서
    - 클라이언트 인증서

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d362329c-3f77-4034-bcf2-a17abd5ce9cc/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5311cc66-44b0-4551-acda-01f56aa7e6b2/Untitled.png)

### kube-api server

- API 서버는 kubernetes 클러스터를 관리하기 위해 다른 구성요소 또는 외부 사용자에게 HTTPS 서비스를 노출한다.
- kube-apiserver은 서버이고, 클라이언트와의 모든 통신에서 보호하기 위해 인증서가 필요하다.
- 그래서 우리는 인증서와 키 쌍을 생성한다.
    - APIserver.crt, APIserver.key

### etcd server

- 클러스터에서 다른 서버는 ETCD 서버이다.
- ETCD 서버는 클러스터에 대한 다양한 정보가 저장되어 있다.
- 그래서 그것은 인증서와 키 쌍이 필요하다.
    - etcdserver.crt, etcdserver.key

### kubelet server

- 워커 노드 안에 있는 서비스이다.
- 이것 또한 kube-api server와 워커 노드의 상호작용을 위해 HTTPS API 엔드 포인트를 노출한다.
- 인증서와 키 쌍이 필요하다.
    - kubelet.crt, kubelet.key

## 클라이언트 구성 요소

### admin

- kube-apiserver에 접속하는 클라이언트는 kubectl을 통해 API를 접속하는 관리자이다.
- 관리 사용자는 Kube-API 서버에 인증하기 위한 키와 인증서 쌍이 필요하다.
    - admin.crt, admin.key

### scheduler

- 스케줄러는 스케줄링이 필요한 파드를 찾고  kube-apiserver와 통신하고, API 서버를 가져와 올바른 노드에 파드를 예약한다.
- 스케줄러는 kube-apiserver에 액세스하는 클라이언트이다.
- 따라서 클라이언트 TLS 인증서를 사용한다.
    - scheduler.crt, scheduler.key

### kube controller

- Kube-apiserver에 액세스하므로 클라이언트이다.
- 따라서 kube-apiserver에 인증을 하기 위한 인증서 키 쌍이 필요하다.
    - controllermanager.crt, controllermanager.key

### kube-proxy

- kube-apiserver에 인증하기 위해 인증서 및 키 쌍이 필요하다.
    - kubeproxy.crt, kubeproxy.key

### kube-api server

- etcd와 통신하기 위해서, kube-api server은 클라이언트가 된다.
- kubelet과 통신하기 위해 kube-api server은 클라이언트가 된다.
    - 기존에 생성해둔 인증서 및 키 쌍을 사용하거나 새로 생성할 수 있다.

## CA

- 쿠버네티스 클러스터 내부에는 클러스터에 대한 인증 기관이 하나 이상 있다.
- 그리고 인증 기관 자체 인증서 및 키가 있다.
    - ca.crt, ca.key

---

# 145. Kubernetes TLS - Certificate Creation

- 인증서를 생성하기 위해 Easy-RSA, OpenSSL, CFSSL 등 다양한 도구를 사용할 수 있다.
- OpenSSL을 이용하여 인증서를 생성해보자.

### CA 인증서

- 개인키를 만든다.

```bash
openssl genrsa -out ca.key 2048
```

- 인증서 서명을 요청한다.

```bash
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
```

- 인증서에 서명한다.
    - CA 자체를 위한 것이기 때문에
    - 자체 개인 키를 사용하여 CA에서 자체 서명한다.

```bash
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crs
```

CA에는 개인키와 경로 인증서 파일이 있다.

### 클라이언트 인증서

- admin user
    - 개인키를 생성한다.
    
    ```bash
    openssl genrsa -out ca.key 2048
    ```
    
    - crs을 생성하고, admin user의 이름을 지정한다.
    
    ```bash
    openssl req -new -key admin.key -subj\
    "/CN=kube-admin/O=system:master" -out admin.csr
    ```
    
    - CA 개인키와 인증서로 서명된 인증서를 생성한다.
    
    ```bash
    openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
    ```
    
    **다른 사용자와 어떻게 구별할까?**
    
    - 사용자 계정을 식별해야 한다.
    - 따라서 그룹을 파라미터로 세부 정보에 추가하면 된다.
    
    ```bash
    "/CN=kube-admin/O=system:master"
    ```
    
- 위의 과정을 통해 kube-apiserver에 액세스하는 모든 클라이언트 컴포넌트에 대한 인증서를 생성할 수 있다.
- 쿠버네티스에서 다양한 컴포넌트가 서로를 확인하려면 CA의 루트 인증서 사본이 필요하다.

### 서버 인증서

- ETCD 서버
    - ETCD 서버에 대한 인증서를 생성한다.
        - 이름은 etcd-server로 지정한다.
        - ETCD는 고가용성을 위해 여러 서버에 걸쳐 클러스터를 구성하는데, 이러한 경우 안전한 통신을 위해 클러스터의 서로 다른 구성원 간 추가 피어 인증서를 생성해야 한다.
    - 인증서가 생성되면 ETCD 서버가 시작되는 동안 파일로 지정할 수 있다.
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d9d85bf6-b8ee-4092-983e-aef7add2c4b9/Untitled.png)
        
- Kube API 서버
    - 클러스터 내에서 많은 이름과 별칭으로 사용된다.
        - 따라서 kube API 서버용으로 생성된 인증서에는 해당 이름이 모두 있어야 한다.
    - 키를 생성한다.
        
        ```bash
        openssl genrsa -out apiserver.key 2048
        ```
        
    - kube-apiserver로 이름을 지정해서 인증서를 생성을 요청한다.
        
        ```bash
        openssl req -new -key apiserver-key -subj\
        "/CN=kube-apiserver" -out apiserver.csr --config openssl.cnf
        ```
        
        **그렇다면 다른 이름은 어떻게 대체할까?**
        
        - OpenSSL 구성 파일을 만들어야 한다.
        - 대체 이름을 지정한다.
        - API 서버가 통과하는 모든 DNS 이름과 IP 주소 등을 포함한다.
        - 그리고 이 구성 파일을 옵션으로 전달한다.
    - 인증서에 서명하여 인증서를 생성한다.
        
        ```bash
        openssl x509 -req -in apiserver.csr\
        -CA ca.crt -CAkey ca.key -out apiserver.crt
        ```
        
    - 키를 지정할 위치
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c24b70ee-66b1-4633-8a99-662ca92532e1/Untitled.png)
        
        - etcd나 kubelet에서 클라이언트 위치에서 인증서가 전달되어야 한다.
        - 따라서 클라이언트 인증서를 지정헤애 힌다.
- kubelet 서버
    - 각 노드의 이름을 따서 인증서 이름이 지정된다.

---

# 146. View Certificate

- 클러스터가 어떻게 세팅되었는지 알아야한다.
    - 클러스터를 처음부터 배포하는 경우
        - 모든 인증서를 직접 생성하는 방법
    - kubeadm을 이용하여 자동화 프로비저닝한 방법

### kubeadm을 이용하여 프로비저닝한 경우

- 사용된 인증서 파일 목록 만들기
- 해당 정보를 얻기 위해 사용된 인증서 파일로 시작한다.
- api 서버를 시작하는데 사용되는 명령 정보가 있다.

```bash
cat /ect/kubernetes/manifests/kube-apiserver.yaml
```

- 해당 경로를 인증서 파일을 기록해둔다.
- 그리고 인증서에 대한 자세한 정보를 찾는다.
- 파일을 입력으로 사용하고 인증서를 디코딩하여 세부 정보를 본다.

```bash
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noou
```

---

# 150. Certificate API

- 새로운 관리자가 들어왔다.
- 우리는 새 관리자가 클러스터에 액세스할 수 있도록 인증서와 키 쌍을 가져와야 한다.

### CA

- CA는 우리가 생성한 키와 인증서 파일의 쌍이다.
