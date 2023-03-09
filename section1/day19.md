## DNS

- 쿠버네티스는 클러스터 설정 시 default로 내장 DNS 서버를 배포한다.
- 서비스가 생성되면 쿠버네티스 DNS 서비스는 서비스에 대한 레코드를 생성한다.
    - 서비스 주소와 IP 주소를 매핑하는 A레코드 생성
    - 따라서, 클러스터 내 모든 파드는 해당 서비스 이름을 사용해 서비스에 도달할 수 있다.
- 같은 네임스페이스 내에 있으면 이름으로 서로 부르고, 다른 네임스페이스라면 전체 이름을 사용한다.
- 각 네임스페이스에 대해 DNS 서버는 하위 도메인을 만든다.
- 모든 서비스는 svc라는 하위 도메인으로 그룹화되어 있다.
- (서비스이름).(네임스페이스).svc → **webservice.apps.svc**
- 모든 서비스의 파드는 default로 cluster.local로 설정된다.
- **webservice.apps.svc.cluster.local**

---

- 중앙 DNS 서버에 IP 항목을 지정해두고, 각 호스트(파드) /etc/resolv.conf 파일에 네임 서버에 DNS 서버 IP를 지정해둔다.
- 쿠버네티스에서 클러스터 내 DNS 서버가 바로 Core DNS 서버이다.

### Core DNS

- 쿠버네티스 클러스터에서 kube-system에서 파드 형태로 배포된다.
- 파드는 CoreDNS를 실행한다.
- /etc/coredns에 위치한 Core File을 사용한다. 이 파일은 여러 플러그인으로 구성되어 있다.
- 클러스터의 최상위 도메인 이름이 설정된다.
- 새로운 서비스 또는 파드를 감시하고, 생성되면 데이터베이스에 레코드를 추가한다.
- 파드가 생성되면 자동으로 DNS 서버와 IP 주소가 자동으로 생성된다. → kubelet이 수행

## Ingress

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6ca8b15b-21b8-4b6b-9648-798e979c8c62/Untitled.png)

- 사용자의 URL 경로를 기반으로 클러스터 내의 다른 서비스로 라우팅하도록 구성할 수 있다.
- 외부에서 액세스 가능한 단일 URL을 사용하여 애플리케이션에 액세스할 수 있도록 도와준다.
- 또한 SSL 보안도 구현할 수 있다.
- 쿠버네티스 클러스터에 내장된 L7 로드 밸런서와 같은 역할을 한다.

### 작동방식

- 인그레스 컨트롤러에서 모든 로드 밸런싱, 인증, SSL, URL 기반 라우팅을 수행한다.
- 지원하고자하는 솔루션을 배포(ingress controller)하고, rule 집합을 지정(ingress 리소스)하여 ingress를 구성한다.
- 쿠번네티스는 default로 ingress 컨트롤러를 지원하지 않으므로, 배포해야한다.

### ingress controller

- Ingress에서 사용할 수 있는 솔루션은 7가지이다.
    - GCE, NGINX, Contour, HAProxy, Traefik, Astel
- 쿠버네티스에서 NGINX Ingress 컨트롤러를 사용해보자.
    - ingress 이미지 배포, 이를 expose하는 서비스, nignx config 데이터 제공 config map, 오브젝트에 액세스할 수 있는 롤이 있는 서비스 계정 배포

### Ingress resources

- 들어오는 모든 트래픽을 라우팅하도록 rule을 구성한다.
- 각 호스트 또는 도메인 이름에 대한 rule이 맨 위에 있고, 각 rule 내에는 URL을 기반으로 트래픽을 라우팅하는 서로 다른 파드가 있다.
