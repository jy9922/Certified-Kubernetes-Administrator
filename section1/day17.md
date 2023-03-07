# Networking Pre-Requisites

### Switching

- 스위치에 연결하려면 호스트에 따라 물리적 또는 가상의 호스트에 인터페이스가 필요하다.
- 호스트의 인터페이스를 보려면 ip link 커맨드를 사용한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bea1c997-7400-4ec8-b111-53b762186cbd/Untitled.png)

- 스위치에 연결할 때 사용되는 eth0 인터페이스가 있다.
- ip addr 커맨드로 동일한 네트워크에 있는 IP 주소를 시스템에 할당한다.
- IP 주소가 할당되면 컴퓨터는 스위치를 통해 통신할 수 있다.
- 스위치는 같은 네트워크 내에서 통신할 수 있다.

### Routing

- 서로 다른 네트워크에서 통신을 하려면 라우터를 거쳐야 한다.
- 라우터에는 두 개의 IP가 할당된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6b73a301-6cd3-46bb-9401-02c9a271efcf/Untitled.png)

### Gateway

- ip route add 커맨드로 문(게이트웨이)를 생성하고, 이를 통해 다른 네트워크 영역까지 도달할 수 있도록 한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f65092c4-9a57-4e1d-bab7-6dc3313d5bb2/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1e647b78-0ec8-436b-8474-b07fb34f53d7/Untitled.png)

### default Gateway

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/744f8e6b-ecb4-4df7-bc4c-a22632af269f/Untitled.png)

- 0.0.0.0 → 모든 IP 대상을 의미한다.

### Linux Route

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/996a8a33-1c7e-43ab-85dc-b464c89c525f/Untitled.png)

- A에서 C로 ping을 날리기 위해서는 A에는 C를 가기 위해 B로 가야되는 정보가, C에는 A에 가기 위해 B로 가야되는 정보가 있어야한다.
- Linux는 default로 패킷이 한 인터페이스에서 다른 인터페이스로 전달되지 않는다.
- 하지만 다음 예제의 경우 두 인터페이스 모두 사설 네트워크에 연결되어 있으므로 둘 사이가 보안상으로 안전하므로 인터페이스로 패킷이 전달된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/754f788e-343b-40be-bcb3-e2bae1d420a1/Untitled.png)

```python
cat /proc/sys/net/ipv4/ip_forward
```

- /proc/sys/net/ipv4/ip_forward에서 시스템의 설정에 따라 결정된다.
- default로는 이 파일의 값은 0으로 설정되어 있으며, 이는 전달이 없음을 의미한다.
- 이 값을 1로 설정하면 ping이 통과할 수 있게된다.
    - 하지만, 재부팅하면 계속 지속되지는 않는다.
    - 지속하려면 /etc/sys/control.conf 파일을 수정해야 한다.

```python
echo 1 > /proc/sys/net/ipv4/ip_forward
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0dfa1c33-4462-48bc-bc12-6b156d4d0a1a/Untitled.png)

- ip link : 호스트의 수정 인터페이스 나열
- ip addr add : 해당 인터페이스에 할당된 IP 주소 확인
    - 재부팅하면 없어짐
    - 영구적으로 유지하려면 /etc/network/interfaces에서 설정해야 함
- ip route add : 라우팅 테이블에 항목 추가
- cat /proc/sys/net/ipv4/ip_forward로 IP 전달 활성화 확인

# Linux DNS
### Name Resolution

- 동일한 네트워크에 속하는 두 대의 컴퓨터끼리 DNS를 가지고 통신하려고 한다.
- 따라서 시스템 A는 시스템 B가 ‘db’라는 것을 알아야 한다.
- 그러기 위해서는 /etc/hosts 파일에 IP와 이름 항목을 추가한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/24546907-3937-4f6d-8ebd-dfaa3f3c0222/Untitled.png)

- 시스템 A가 db에 ping을 날리면 가는 것을 확인할 수 있다.
- /etc/hosts 파일에서 원하는 만큼의 서버에 대한 원하는 만큼의 이름을 가질 수 있다.

### DNS

- 인터넷이 방대해지면서 파일과 항목을 관리하기 위해 중앙에서 관리할 단일 서버를 두게된다.
- 이를 DNS 서버라고 한다.
- /etc/hosts 파일 대신 DNS 서버를 통해 IP를 찾게된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d2550e40-968e-4555-ae27-5bc6995ce4b6/Untitled.png)

- DNS 서버로 지정해보자.
- DNS 서버의 IP는 192.168.1.100이다.
- 모든 호스트에는 /etc/resolv.conf에 DNS resolution configuration 파일이 있다.
- 여기에 DNS 서버 주소를 지정하면 된다.
    - nameserver 192.168.1.100
- 이렇게 하면 각 호스트는 자신이 알지 못하는 도메인이 발견되면 해당 네임서버(DNS서버)에서 IP 주소를 조회한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3c9cb4e4-3c17-40da-b277-093727e51062/Untitled.png)

- 같은 주소에 대한 IP 정보가 /etc/host와 네임서버에 둘 다 있으면 어떤 순서로 확인할까?
- 보통은 /etc/hosts 파일을 먼저 확인하는데, 그 순서를 etc/nsswitch.conf 파일의 항목으로 정의한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5cfdae6a-4e04-4159-aa36-4f09dae053fc/Untitled.png)

- 위는 files 후 dns이기 때문에 /etc/hosts 파일 → DNS 서버가 된다.
- 목록에 없는 서버에 ping을 시도하면 어떻게 될까?

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/615faec9-09e3-4ff5-87c0-3038ab158d48/Untitled.png)

- resolve.conf 파일에 다른 항목을 추가하여 facebook을 알고있는 네임서버를 가리킬 수 있다.
- 예를 들어, 8.8.8.8과 같은 Google에서 호스팅하는 인터넷에서 사용할 수 있는 널리 알려진 공용 네임 서버를 주소를 설정할 수 있다.

### Domain Name

- 트리 구조로 이루어져 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7d7d9e7c-ec96-4549-86ac-81e30ad78e9f/Untitled.png)

- .com → Google 서버
- IP를 찾으면 DNS 서버는 일정 시간 IP 캐싱

### Record Types

- 호스트 이름에 IP를 저장한다. A 레코드
- 한 이름에 다른 이름을 매핑하는 것은 CNAME 레코드이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2eb5cb4f-83a8-4424-9496-bc74e12e04bf/Untitled.png)

### Networking Tool

- ping
- nslookup : DNS 서버에서 호스트 이름 쿼리 (/etc/hosts 파일 항목 고려하지 않음, DNS 서버에만 쿼리함)
- dig : DNS 이름 확인 테스트

---

## CoreDNS

- DNS 서버로 구성하는 방법
- DNS 서버 전용 서버와 서버의 항목으로 구성할 일련의 IP가 제공된다.
- Github 릴리즈 페이지에서 도커 이미지로 다운받을 수 있다. ( 혹은 바이너리 파일 다운 가능 )
- 실행 파일로 DNS 서버로 시작한다.
- defualt로 DNS 서버의 기본 포트인 53에서 수신 대기한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4addf383-925b-47f4-ad55-085b2856048e/Untitled.png)

# Linux Namespace
리눅스 네임스페이스는 Docker와 같은 컨테이너에서 네트워크 격리를 구현하는데 사용된다.

### Process Namespace

- 컨테이너를 생성할 때 컨테이너가 격리되어 있는지 확인하고, 호스트의 다른 프로세스나 다른 컨테이너를 볼 수 없도록 해야한다.
- 컨테이너 안에서는 컨테이너 자신이 실행하는 프로세스만 보고 자체 호스트에 있다고 생각한다.
- default 호스트는 컨테이너 내부에서 실행되는 프로세스를 포함한 모든 프로세스를 볼 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cbb7bdae-5be9-4fa6-a849-3c88051b9882/Untitled.png)

### Network Namespace

- 호스트에는 로컬 영역 네트워크에 연결하는 자체 인터페이스가 있다.
- 호스트에는 나머지 네트워크에 대한 정보가 포함된 자체 라우팅 및 ARP 테이블이 있다.
- 컨테이너가 생성되면 네트워크 네임스페이스를 생성하기에 호스트의 네트워크 정보를 볼 수 없다.
- 네트워크 네임스페이스 내에서 컨테이너는 자체 가상 인터페이스, 라우팅 및 ARP 테이블을 가질 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d0bd9d21-6d26-46ff-88aa-9b41d63dbeeb/Untitled.png)

- ip netns add로 namespace를 만들고, ip nets로 조회한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c4f7ffda-80a2-4bd6-a5cb-854407ddd5dc/Untitled.png)

- 이후 각각 네임스페이스에서 호스트 인터페이스를 조회해보면 밖의 호스트 인터페이스는 볼 수 없는 것을 확인할 수 있다.

### ARP and Routing Table

- ARP과 라우팅 테이블 또한 해당 네임스페이스에 들어가서 보면 나타나지 않는다.

### Virtual Cable

- 네임스페이스에 가상의 인터페이스를 만들어 두 개의 네임스페이스를 연결할 수 있다.
- 파이프라고 한다.
- veth로 설정하고, ip link at 커맨드를 실행하고 두 끝 veth red와 veth blue를 생성한다.
- 각 인터페이스를 적절한 네임스페이스에 연결한다.
    - ip link set veth red netns red

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bae7651a-1f2d-4f4c-b4b9-0139210e6957/Untitled.png)

- 각 네임스페이스에 IP주소를 할당한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/14d8085a-ab83-43eb-a782-168253e965de/Untitled.png)

- ip addr 커맨드를 사용하여 IP 주소를 할당하지만 각 네임스페이스 내에서 사용한다.
- 각 네임스페이스 내의 각 장치에 ip link set up을 통해 인터페이스를 불러온다.
- link를 통해 연결한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0d80c690-bf28-44b6-b8d9-b5ac6979b435/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b521e233-fc86-47a8-88cb-3a57951c7050/Untitled.png)

- 호스트 ARP 테이블은 새 네임스페이스에 대한 내용은 전혀 모른다.

### Linux Bridge

- 네임스페이스가 더 많으면 의사소통을 위해 호스트 네부에 가상 네트워크를 만든다.
- 네트워크를 생성하려면 스위치가 필요하므로 가상 스위치가 필요하다.
- 호스트 내에 가상 스위치를 생성하고, 여기에 네임스페이스를 연결한다.
- 호스트 내에 가상 스위치는 Linux 브리지 옵션을 사용할 수 있다.
- ip link add 를 사용하여 호스트에 새 인터페이스를 추가한다. (v-net-0)
- ip link set up으로 스위치를 켠다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b8375c90-8a54-469d-9c86-7387814ee100/Untitled.png)

- 네임스페이스를 새로운 가상 네트워크 스위치에 연결하자.
- 모든 네임스페이스를 브리지 네트워크에 연결한다.
- 네임스페이스를 브리지에 연결하는 새 케이블을 만든다.
- ip link add를 이용해 veth red와 veth-red-br을 생성한다. blue도 마찬가지이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fd32e981-a178-45ce-bd5c-054f25e04d4a/Untitled.png)

- 케이블이 준비되면 네임스페이스에 연결한다.
- ip link set veth-red netns red로 red 인터페이스에 연결한다. 그리고 ip link set veth-red-br master v-net-0 으로 연결한다. blue 또한 연결해준다.
- 링크에 대한 IP를 활성화하고, 장치를 켠다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fc36394d-ce95-4620-8b84-a8a91ce2c3f0/Untitled.png)

- 컨테이너는 네트워크를 통해 서로 도달할 수 있다.
- 네임스페이스에서 호스트 인터페이스에 도달할 수 있을까?
- 아니다. 왜냐? 서로 다른 네트워크 대역에 있기 때문이다. 만약 둘을 연결시키고 싶다면, 가상 스위치 또한 호스트의 또다른 인터페이스이므로 가상 스위치에 네임스페이스와 같은 대역의 IP를 할당해주면 된다.
- 호스트 → 네임스페이스 ping이 가능하다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/54833a35-2d09-417a-98c7-a424371e2673/Untitled.png)

- 여기서 외부 세상과 통신은 오직 eth0만 가능한 것을 잊지말아야 한다.
- 따라서 네임스페이스에서 외부 IP로 ping을 날려도 보내지지 않는다.
- 이를 해결하기 위해서는 라우팅 테이블을 추가해야 한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/07daceb6-8699-4c61-ab16-4f371326ad62/Untitled.png)

- 게이트웨이는 바로 v-net-0이 된다.
- 따라서 blue 네임스페이스에서 192.168.15.5 게이트웨이를 통해 모든 트래픽을 192.168.1로 향하게 라우팅할 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0c0fe994-2c44-4f91-ab69-fc2aed3f23da/Untitled.png)

- 라우터를 통해 외부와 통신을 할 때, 게이트웨이 역할을 하는 호스트에서 NAT 활성화가 필요하다.
- 호스트에 iptables를 이용해 포스트 라우팅 체의 NAP IP 테이블에 새로운 rule을 추가한다.
- 네트워크에서 보내는 패킷 자체를 192.168.15.0으로 바꾼다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cf21b89d-1b71-4a5b-b0c1-d2e1b8ac2dab/Untitled.png)

- blue 라우팅 테이블에 default 게이트웨이를 추가한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/94bfc4ed-e8bb-411a-8a8f-33e1707f2d01/Untitled.png)

- 외부에서 내부로 들어오기 위해서는 개인 네트워크 ID를 두 번째 호스트에 제공하면 된다.
- 80 포트로 들어오는 요청은 DNAT로 들어오게 iptables를 업데이트해준다.
- 또는 로컬 호스트에 포트 80으로 들어오는 트래픽이 blue 네임스페이스에 할당된 IP의 포트로 전달되도록 iptables를 사용하여 룰을 추가할 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/283131bd-ffd1-47ea-b4d7-ebde8a8fde08/Untitled.png)

# Docker Networking
- Docker가 설치된 서버인 단일 Docker 호스트를 살펴보자.
- eth0에는 IP주소가 로컬 네트워크에 연결하는 인터페이스가 있다.

### None

- 네트워킹 옵션이 none 이라면, 도커 컨테이너는 네트워크에 연결되지 않는다.
    - 따라서, 컨테이너는 외부 세계와 통신할 수 없게 된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/15db1482-13e2-4280-863c-5ea5b324895f/Untitled.png)

### Host

- 컨테이너가 호스트 네트워크에 연결된다.
- 호스트와 컨테이너 간에 네트워크 분리가 없다.
- 호스트의 포트 80에서 바로 컨테이너에 올라간 앱 애플리케이션을 사용할 수 있다.
- 동일한 컨테이너의 다른 인스턴스를 실행하려고 하면 호스트 네트워크를 공유하기 때문에 작동하지 않는다.
    - 두 프로세스에서 동일한 포트에서 동시에 수신할 수 없다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eaf6642f-58e8-4142-909c-5b8914e1bb0b/Untitled.png)

### Bridge

- 내부 전용 네트워크가 생성된다.
- 네트워크 기본 주소로 172.17.0.0이 있고, 네트워크에 연결된 컨테이너는 자체 내부 전용 네트워크 주소를 가진다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1ef32d97-2e35-4c51-b334-e09ea5050d84/Untitled.png)

- 호스트에 Docker를 설치하면 기본적으로 bridge라는 내부 전용 네트워크가 생성된다.
- docker network ls를 통해 확인할 수 있다.
- 호스트에서는 Docker0라는 이름으로 생성된다.
- Docker0에는 172.17.0.1이 할당된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bc3d4ba8-84b8-407b-8a96-2ba64c68a3e9/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/df7a3470-e99f-44f4-a6f1-0daede950489/Untitled.png)

- 컨테이너가 생성될 때마다 Docker는 컨테이너에 대한 네트워크 네임스페이스를 생성한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fee44695-8794-408c-ba05-556fde5e3b22/Untitled.png)

- Docker는 컨테이너 또는 네트워크 네임스페이스를 브리지 네트워크와 어떻게 연결할까?
- 양 쪽 끝에 두 개의 인터페이스가 있는 가상 케이블을 만든다.
- Docker는 호스트에서 IP link 명령을 실행하여 로컬 브리지인 Docker0에 연결된 인터페이스의 한 쪽이 표시된다.
- ip -n <네임스페이스 이름> link를 하면 컨테이너 인터페이스 정보를 볼 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0f54c356-8f58-4525-aa93-ac705df9c0d8/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3ee4fe70-4d69-44ba-b437-05290a29e537/Untitled.png)

- addr을 통해 각각의 인터페이스의 IP 정보도 조회할 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5f13dabe-cc45-4ddd-86aa-81290d185443/Untitled.png)

- 컨테이너가 생성될 때마다 동일 작업을 반복한다.
    - 도커는 네임스페이스를 만든다.
    - 인터페이스 쌍을 만든다.
    - 한 쪽 끝을 컨테이너에, 다른쪽 끝을 Docker0에 연결한다.
    - 인터페이스 쌍은 홀,짝으로 쌍을 이룬다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dba52374-1e22-49f3-a62c-d84f13e397f1/Untitled.png)

- 포트 매핑을 살펴보자.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/592f5ef1-a1d7-4e0d-bf73-0deb82b7ead0/Untitled.png)

- 포트 80의 Docker 호스트 내에서 컨테이너의 IP를 사용하여 curl과 함께 웹 페이지에 액세스하면 웹 페이지가 표시된다.
- 하지만, 호스트 외부에서는 안된다.
- 따라서, 외부에서 접근 가능하게 포트 매핑을 할 수 있다.
- 컨테이너 실행 시, Docker 호스트의 포트 8080을 컨테이너 포트 80에 매핑 시킨다.
    
    ```python
    docker run -p 8080:80 nginx
    ```
    
- 도커는 NAT 테이블에 항목을 생성해서 대상 포트를 8080에서 80으로 변경하는 사전 라우팅 체인에 추가한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f6e9e526-bb69-47be-8f4b-0817838affe9/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/af68ebc2-68ad-4ee3-a2ca-697a9b8bef56/Untitled.png)

# CNI

- 컨테이너 네트워킹 인터페이스
- 컨테이너 런타임 환경에서 네트워킹 문제를 해결하기 위해 프로그램을 개발하는 방법을 정의하는 표준 집합이다.
- 플러그인이 어떻게 개발되어야 하는지, 컨테이너 실행 시간이 어떻게 그것을 호출하는지 정의한다.

# Cluster Networking

### IP & FQDN

- 각 노드에는 네트워크에 연결된 인터페이스가 하나 이상 있어야 한다.
- 각 인터페이스에는 주소가 있어야 한다.
- 고유한 호스트 이름 집합과 고유한 MAC 주소가 있어야 한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9723825c-233f-4278-8627-d9552c63b50e/Untitled.png)

### Ports

- controlplane의 다양한 컴포넌트에 의해 사용된다.
- Kube API는 6443 포트, ETCD 2379 포트, Kubelet은 10250 포트, Kube Controller Manager은 10257 포트, Kube-Scheduler는 10259 포트가 열려 있어야 한다.
- 워커노드는 30000-32767에서 외부 액세스를 위한 서비스를 노출하므로 열려 있어야 한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0238078d-886b-428e-944c-92b8a48c7b17/Untitled.png)

# Pod Networking

- 쿠버네티스는 모든 파드가 고유한 IP 주소를 가지고 있다.
- 모든 파드는 해당 IP 주소를 사용하여 동일한 노드 내의 다른 모든 파드에 도달할 수 있다.
- IP 주소는 자동으로 할당된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ce51c3a7-708b-4ffb-a285-dbb785471d58/Untitled.png)

- 3개의 노드 클러스터가 있다.
- 노드는 외부 네트워크의 일부이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/64136516-cf26-4559-b0f3-788b1c4fd055/Untitled.png)

- 컨테이너가 생성되면 쿠버네티스는 컨테이너에 대한 네트워크 네임스페이스를 생성한다.
- 이들간 통신을 위해, 네임스페이스를 네트워크에 연결하기 위해 노드의 브리지 네트워크에 연결한다.
- 따라서, 노드 내에 브리지 네트워크를 만들고 주소를 부여한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6246990f-8e76-490a-9018-e5fdc2dd2cae/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7f7e2968-2ccd-4f7a-b5fb-7d89293fc9fc/Untitled.png)

- 컨테이너가 생성되면 다음과 같은 작업을 반복한다.
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4fe7f38d-85e1-4196-9a72-be674cd86de9/Untitled.png)
    
    - 가상 네트워크 케이블 쌍 생성 후 컨테이너와 브리지 네트워크에 부착
    - 이후 각각의 케이블에 ip를 할당
    - 게이트웨이에 라우트 추가
- 이 작업을 각 컨테이너에 실행하면, 노드 내의 파드끼리 서로 통신이 가능하다.
- 다른 노드로의 통신은 default 게이트웨이에 라우팅 경로를 추가한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b0042516-6d60-479f-8ef5-90d1f77fa86e/Untitled.png)

- 이를 쿠버네티스에서 파드가 생성될 때마다 자동으로 스크립트를 CNI를 통해 실행시킬 수 있다.
- CNI는 컨테이너를 생성하는 즉시 스크립트를 호출하는 방법을 쿠버네티스에 알려준다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/71b5ee45-2191-46b6-b79d-45b5e2e22c01/Untitled.png)



