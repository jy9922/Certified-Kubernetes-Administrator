# Day01

# 130. Backup and Restore

## Backup

- 쿠버네티스 클러스터 백업시 고려사항
    - Resource Configuration (구성파일)
    - 클러스터와 관련된 정보가 저장되는 ETCD 클러스터
    - 영구 스토리지

### Resource Configuration

- 단일 폴더 안에 모든 개체가 정의되어 있기 때문에
    - 나중에 **쉽게 재사용할 수 있고, 공유가 가능**하다.
- **원본 파일에 대한 복사본이 항상 저장된 상태로 있어야 한다.**
- 가장 좋은 방법은 **GitHub와 같은 소스 코드 저장소에 저장**하는 것!
    - 소스 코드 리포지토리는 올바른 백업 솔루션이 있어야 한다.
    - GitHub와 같은 관리형, public형 소스 코드 리포지토리는 위에 대한 솔루션을 제공한다.
    - 따라서 전체 클러스터를 손실하더라도, 구성파일을 가지고 있으므로 클러스터에 애플리케이션을 재배포할 수 있다.
- **만약 명령형 방식으로 객체를 생성한다면?**
    - 리소스 구성을 백업하는 방식은 **Kube API 서버에 쿼리**하는 방식이다.
    - **kubectl을 사용하여 Kube API 서버에 쿼리**하거나 **직접 API 서버에 액세스 한다**.
    
    ```python
    kubectl get all -all-namespaces -o yaml > all-deploy-services.yaml
    # 출력을 yaml 형식으로 추출하고 해당 파일을 저장한다.
    ```
    
    - 하지만 위 방법은 일부 리소스 그룹에 한정된다.
- 다른 리소스 그룹의 경우에는 툴을 사용하는 방법이 있다.
    - Ark, Heptio, Velero
    - 위 도구는 쿠버네티스 API를 이용해 쿠버네티스 클러스트의 백업을 수행하는데 도움이 된다.

### ETCD

- ETCD 클러스터는 클러스터 상태에 대한 정보를 저장한다.
- 클러스터에서 생성되는 노드 및 기타 모든 리소스에 대한 정보는 ETCD에 저장된다.
- 따라서 리소스 백업 대신 ETCD 서버 자체를 백업하는 방법이 있다.

- ETCD는 마스터 노드에 호스팅된다.
- ETCD를 구성하는 동안 모든 데이터가 저장되는 데이터 디렉토리 위치를 지정할 수 있다.
- 해당 디렉토리가 백업 도구로 백업할 수 있게 구성될 수 있다.

- ETCD 내부에는 snapshot 솔루션이 내장되어 있다.
- ETCD 데이터베이스의 스냅샷을 만들 수 있다.
- ETCD 제어 명령어로 스냅샷 저장 명령을 사용할 수 있고, 스냅샷 이름은 snapshot.db로 지정한다.

```python
ETCDCTL_API=3 etcdctl \
	snapshot save snapshot.db
```

```python
# 현재 디렉토리에서 지정한 이름으로 스냅샷 파일이 생성된다.
ls
snapshot.db

# 만약 다른 위치에 생성하고 싶다면, 전체 경로를 지정하면 된다.

# 백업 상태를 볼 수 있다.
ETCDCTL_API=3 etcdctl \ 
snapshot status snapshot.db
```

## Restore

- kube API 서버 중지시키기

```python
service kube-apiserver stop
```

- ETCD 클러스터를 다시 시작하고, Kube API 서버는 이에 의존한다.
- ETCD 제저 명령어를 스냅샷을 복원한다.

```python
ETCDCTL_API=3 etcdctl \
snapshot restore snapshot.db \
--data-dir /var/lib/etcd-from-backup
```

- ETCD가 백업에서 복원할 때 새 클러스터 구성을 초기화 한다.
    - 실수에 기존 클러스터에 join 되지 않도록 하기 위해서다.
- 이 명령어를 실행하면 새 데이터 디렉토리가 생성된다.
- ETCD 구성 파일을 구성한다.

```python
--data-dir=/var/lib/etcd-from-backup
```

- 데몬을 다시 로드하고, ETCD 서비스를 재시작한다.

```python
systemctl daemon-reload
service etcd restart
```

- 이후 Kube API 서버를 재시작한다.

```python
service kube-apiserver start
```

- 모든 ETCD 명령을 사용하여 인증서 파일을 저장하는 것을 잊지 않아야 한다.
- 인증을 위해 ETCD 클러스터에 엔드포인트 및 CA 인증서, etcd 서버 인증서 및 키를 지정해야 한다.

```python
ETCDCTL_API=3 etcdctl \
snapshot save snapshot.db \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/etcd/ca.crt \
--cert=/etc/etcd/etcd-server/crt \
--key=/etc/etcd/etcd-server.key
```

---

# 131. ETCDCTL

- ETCD는 마스터 노드에서 static pod 형태로 배포된 키-값 데이터베이스 저장소이다.
- 버전은 v3을 사용한다.
- 백업과 복원을 위해 etcdctl을 사용하려면 ETCDCTL_API=3을 세팅해야 한다.

- etcdctl을 사용하기 위해 사전에 ETCDCTL_API를 export할 수 있다.

```bash
export ETCDCTL_API=3
etcdctl version

# 해당 명령어 help 보기
etcdctl -h

# etcd 스냅샷하는 방법
etcdctl snapshot save -h

# etcd 스냅샷 복원 방법
etcdctl snapshot restore -h

```

---

# 132. Backup and Restore

- export ETCDCTL_API=3을 먼저 실행해 ETCD 백업 명령을 사용할 수 있다.

```bash
export ETCDCTL_API=3
```

- ETCD 버전 확인

```bash
# ETCD 버전 확인
ETCD 이미지 버전 확인
```

- /etc/kubernetes/manifests에는 static pod로 배포되는 클러스터 구성 요소 정의 파일 존재

```bash
ls /etc/kubernetes/manifests
etcd.yaml kube-apiserver.yaml kube-controller-manager.yaml kube-scheduler.yaml
```

- etcdctl snapshot 사용시 반드시 옵션을 사용해야 한다.
    - 옵션에 대한 정보는 kubectl describe 명령을 통해 ETCD 파드에서 얻어오기

```bash
etcdctl snapshot save --endpoints=127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
/opt/snapshot-pre-boot.db
```

- 데이터가 복구될 디렉토리를 옵션으로 지정한다.

```bash
etcdctl snapshot restore --data-dir /var/lib/etcd-from-backup /opt/snapshot-pre-boot.db
ls /var/lib/etcd-from-backup/

# etcd 정의 파일에서 동일한 위치를 가리켜야 한다.
vi /etc/kubernetes/manifests/etcd.yaml
```

---

# 136. Security

- TLS 인증서를 사용하여 보호된다.
- 쿠버네티스 관리자는 직접 클러스터를 설정한다. 그리고 인증서와 관련된 문제를 직면하게 된다.

---

# 137. Security Primitives

- 클러스터 자체를 구성하는 호스트에 대한 모든 액세스는 보안되어야 한다.
    - 비밀번호 기반 인증은 불가능하고, SSH 키 기반 인증만 사용할 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6055581d-1f61-46f2-ae63-d33901959ee5/Untitled.png)

### **클러스터를 보호하기 위한 조치에는 무엇이 있을까?**

- 쿠버네티스 모든 작업 내에는 kube API 서버가 중심에 있다.
- kubectl을 통해 상호작용하거나 API에 직접 액세스하여 모든 작업을 수행한다.
- 그것을 통해 클러스터에서 거의 모든 작업을 수행할 수 있기에 이것이 1차 방어선이다.

### API 서버 자체에 대한 액세스 제어

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b3ae3524-af20-4814-859e-9086051e8c40/Untitled.png)

- 두 가지의 결정을 내려야 한다.
    - 누가 클러스터에 액세스 하는가?
    - 무엇을 할 수 있는가?
- 누가 클러스터에 액세스 하는가는 **인증 매커니즘(Authentication)**에 의해 정의된다.
    - API 서버에 인증할 수 있는 방법은 다양하다.
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/460b9ffa-cde2-4c66-bc48-c0e4091880ae/Untitled.png)
        
- 무엇을 할 수 있는가에 대해서는 **권한 부여(Authorization) 매커니즘**에 의해 정의된다.
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ff7913c9-0f6d-4e74-8959-12e9dd1f057c/Untitled.png)
    
- 클러스터의 구성 요소 간 모든 통신은 (마스터 노드 구성요소 부터 kubelet과 kube-proxy와 같이 워커 노드에서 실행되는 모든 것은) TLS 암호화를 사용하여 보호된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8870dcff-6022-429e-9551-81ffdca601f6/Untitled.png)

---

# 139. Authentication

- 다양한 User들은 각자 목적에 맞게 클러스터 내에 접근한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9e41306f-7ca4-4815-8d97-798e7b7029c4/Untitled.png)

- 엔드 유저는 응용 프로그램 자체에 의해 내부적으로 관리된다. 그래서 논외한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c7142e9c-a2e1-4b9e-bf18-eaed7f525d15/Untitled.png)

- 사용자의 쿠버네티스 클러스터의 접근은 관리적인 목적으로 두 타입의 유저로 나눌 수 있다.
    - 사람(User)과 프로세스(Service Account)로 나눌 수 있다.
- 쿠버네티스는 사용자 계정을 관리하지 않는다. 그래서 외부 소스에 의존한다.
    - 따라서 쿠버네티스 클러스터에서는 사용자를 생성할 수 없고, 사용자 리스트를 볼 수 없다.
- 서비스 계정은 쿠버네티스가 관리한다.
    - 쿠버네티스 API를 통해 서비스 계정을 생성하고 관리할 수 있다.
    
    ```bash
    kubectl create serviceaccount sa1
    
    kubectl get serviceaccount
    ```
    

- 모든 사용자의 접근은 Kube API Server에 의해 관리된다.
- kubectl 툴을 통해 클러스터에 접근하든 API를 직접 사용하든 상관없다.
- 모든 요청은 Kube API Server로 간다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/aefbd72d-5d30-433d-a8fd-609083b85f92/Untitled.png)

- Kube API Server은 요청을 처리하기 전에 먼저 인증을 한다.

- 인증하는 방식에는 다양한 방법이 있다.
    - 고정 암호 파일
    - 고정 토큰 파일
    - 인증서
    - 타사 인증 프로토콜

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9c03c2e5-8f48-4fdf-94df-8e44a44d9439/Untitled.png)

### 고정 암호 파일

- csv 파일에 사용자 목록과 암호를 만들고, 사용자 정보의 소스로 사용한다.
- 파일 이름을 Kube API Server 옵션으로 지정하고, 재시작한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/99084baf-e9e4-4a27-95d2-e0ec850d3d0f/Untitled.png)

- kubeadm 툴을 사용했다면 Kube API Server 정의 파일을 수정해야한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c82cbb70-aa1c-4f29-8224-f2f220e92f7d/Untitled.png)

- 사용자 및 암호를 다음과 같이 지정할 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d32b4f41-4a86-45b2-a44a-89746104260d/Untitled.png)

- 정적 토큰 파일 경우 옵션을 다음과 같이 설정하면 된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/539c7078-71bd-4587-af54-6c31bf539cc8/Untitled.png)

고정 파일 방법은 불안정하기 때문에 비추천하는 방법이다.

# 143. TLS Basics

- 인증서는 거래 사이에 신뢰를 보장하기 위해 사용된다.예
- 예를 들어, 사용자와 웹 서버 사이의 통신을 TLS 인증서가 보장한다.
    - 웹 서버가 액세스하려고 할 때 TLS 인증서는 사용자와 서버 간의 통신이 암호화되고 서버가 누구인지 확인한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c346ca98-6fc7-4dbd-9de8-2c9772b56447/Untitled.png)

- 보안 연결이 없으면 사용자가 자신의 온라인 뱅킹 프로그램에 액세스하는 경우 입력한 자격 증명이 일반 텍스트 형식으로 전송된다.
- 따라서, 네트워크 트래픽을 스니핑하는 해커는 자격 증명을 쉽게 검색하고 이를 사용하여 사용자의 은행 계좌를 해킹할 수 있다.

- 대칭키 : 안전한 암호화 방식이지만 데이터를 암/복호화 하는데 같은 키를 사용한다.
    - 따라서, 해커가 중간에 키를 탈취하게 되면 데이터를 복호화활 위험이 있다.
- 비대칭키 : 암호화는 단일 키를 사용하여 데이터를 암호화하고 복호화한다.
    - 공개키와 개인키가 있다.
    - 공개키로 데이터를 암호화하면 해당 개인키로만 데이터를 복호화 할 수 있다.
    - 따라서 개인키는 항상 안전하게 보호받고 있어야 하며, 다른 사람과 공유해서는 안된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f0ca8bfa-2ff6-4e7f-aeae-0c09710dace0/Untitled.png)

- 서버에 공개키/개인키 쌍을 생성한다.

```bash
ssh-keygen
id_rsa  id_rsa.pub # 개인키, 공개키
```

- 공개키로 잠긴 문을 통과하는 경우를 제외하고, 서버에 대한 모든 액세스를 잠가 서버를 보호한다.

```bash
cat ~/.ssh/authorized_keys
```

- SSH를 시도할 때, SSH 명령에서 개인 키의 위치를 지정한다.

```bash
ssh -i id_rsa user1@server1
```

- 환경에 다른 서버가 있는 경우, 공개키의 복사본을 만들어 원하는 만큼 많은 서버에 배치할 수 있다. 동일한 개인키를 사용하여 모든 서버에 안전하게 SSH로 연결할 수 있다.
- 다른 사용자가 서버에 액세스해야 하는 경우, 유저 자체의 공개 및 개인 키 쌍을 생성한다. 해당 서버에 액세스할 수 있는 추가 쌍을 생성하고, 공개키로 잠그고 공개키를 모든 서버에 복사하면 다른 사용자 또한 개인키를 사용하여 서버에 액세스할 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/657fb268-6958-462a-9b4d-5efd0db03f75/Untitled.png)

- 대칭키는 데이터를 암/복호화하는데 사용하는 키가 동일 네트워크를 통해 전송되기 때문에 해커가 해당 키에 접근해 데이터를 해독할 위험이 있다.
- 따라서, 대칭키를 옮기기 위해 비대칭키 암호화 방법을 사용한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8a8f0db9-35a2-4c33-8426-156a27639d2f/Untitled.png)

- 서버에서 공개키 및 개인키 쌍을 생성한다.
- openssl 커맨드를 사용하여 개인키와 공개키 쌍을 생성한다.

```bash
openssl genrsa -out my-bank.key 1024
```

- 사용자가 https를 사용하여 웹 서버에 액세스하면, **서버에서 공개키를 가져온다.**
- 해커가 모든 트래픽을 스니핑하고 있으므로 그도 공개키 복사본을 얻었다 가정해보자.
- **사용자 브라우저는 서버에서 제공하는 공개키를 사용하여 대칭키를 암호화**한다.
- 이후 그것을 **서버에게 보낸다.**
- 서버는 개인키를 사용하여 **메시지를 해독하여 메시지에서 대칭키를 검색한다.**
    - 여기서 해커는 메시지에서 대칭키를 해독하고 검색할 수 있는 개인키가 없다.
    - 해커는 메시지를 잠그거나 암호화할 수만 있고 메시지를 해독할 수 없는 공개키만 가지고 있다.
- **따라서 대칭키는 이제 사용자와 서버에서만 안전하게 사용할 수 있다.**
- 비대칭키를 사용하여 데이터를 암호화하고 서로에게 보낼 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2936cd02-912f-46ad-9829-440a39533e14/Untitled.png)

- 동일한 대칭키를 사용하여 데이터를 해독하고 정보를 검색할 수 있다.
- 따라서 비대칭키를 사용하여 대칭키를 사용자에서 서버로 성공적으로 전송하고, 대칭 암호화를 사용하여 향후 모든 통신을 보호한다.

- 해커 입장에서는 우리의 계정을 해킹할 새로운 방법을 찾는다.
- 바로, 해커가 제시하는 양식에서 우리가 직접 입력하는 방법이다.
- 이를 위해 해커는 웹 사이트와 똑같이 생긴 웹 사이트를 만든다. 그리고 자신의 서버에서 호스팅한다.
- 이후 웹 사이트로 가는 경로를 자신의 서버로 라우팅하기 위해 사용자의 환경과 네트워크를 조정한다.
- 사용자는 의심없이 자신의 패스워드를 입력한다.
- 브라우저는 키를 수신하고 암호화된 대칭키를 보내고, 해당 키로 암호화된 자격 증명을 보내게 된다. 해커는 대칭키로 자격 증명을 해독한다.

**서버에서 받은 키가 실제 웹 사이트에서 보낸 합법적인 키인지 어떻게 알 수 있을까?**

- 서버가 키를 보낼 때는 키가 포함된 인증서를 보낸다.
- 인증서는 디지털 형식으로 이루어져 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/35d13c3d-a1c8-4443-85d1-d8d5f05cf082/Untitled.png)

- 인증서 발급 사람, 서버의 공개 키, 해당 서버의 위치 등에 대한 정보가 있다.
- 그리고 인증서에 이름이 있으며, 인증서가 발급된 사람 또는 주체는 자신의 신원이 확인되는 필드이므로 매우 중요하다.
- 웹 서버인 경우 사용자가 브라우저에 URL에 입력한 내용과 일치해야 한다.
    - 다른 이름이 있는 경우 이름 섹션의 subject 아래에 지정해야 한다.
- 하지만 누구나 이 인증서를 생성할 수 있다.

**따라서, 인증서를 보고 어떻게 합법적인지 확인할 수 있을까?**

- 인증서를 생성한 경우 직접 서명해야 한다. 이를 self-signed certificate라고 한다.
- 따라서 이러한 인증서의 경우에는 안전한 인증서가 아니다!

- 브라우저에는 인증서 유효성 검사 매커님이 내장되어 있다.
- 브라우저 텍스트에서는 서버에서 받은 인증서가 합법적인지 확인하기 위해 유효성 검사를 한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cc1cabc1-5f34-4d3b-a851-42d19bd46074/Untitled.png)

**그렇다면 웹 브라우저가 신뢰할 수 있는 웹 서버에 대한 합법적인 인증서는 어떻게 생성할까?**

- 인증기관 또는 CA가 담당한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bcd1921d-29d7-4c1e-b000-68fe42a04287/Untitled.png)

- 우리를 위해 인증서에 서명하고 검증할 수 있는 조직이다.
- Symatec, DigiCert, Comodo, GlobalSign 등이 있다.
- 작동 방식은 이전에 생성한 키와 웹 사이트 도메인 이름을 사용하여 certificate signing a request or CSR을 생성한다.
- open SSL 커맨드를 사용하여 이 작업을 다시 수행할 수 있다. 위 커맨드는 my-bank.csr file을 생성한다.
- 이 파일은 서명을 위해 CA로 보내야 하는 인증서 서명 요청인 CSR 파일이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e8078f2b-e208-464a-adc6-0ace81fbf35a/Untitled.png)

- 인증 기관은 우리의 세부 정보를 확인한 후 인증서에 서명하고 다시 보낸다.
- 이제 브라우저가 신뢰하는 CA에서 서명한 인증서가 된다.
