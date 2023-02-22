# Day08

# Application Commands

- Docker 컨테이너를 실행한다고 가정해보자.
- Docker run Ubuntu 명령을 실행할 때, Ubuntu 이미지 인스턴스를 실행하고, 즉시 종료한다.

```yaml
docker run ubuntu
```

```yaml
docker ps -a
# 를 입력하면 중지된 것을 포함하여 모든 컨테이너를 보여준다.
```

❓ **왜 그런것일까**

- 가상머신과 달리 컨테이너는 운영체제를 호스트하기 위한 것이 아니다.
- 컨테이너는 프로세스와 같은 특정 작업을 실행하기 위한 것이다.
    
    ( 웹 서버 또는 애플리케이션 서버 또는 데이터베이스 서버 등 )
    
- 컨테이너는 작업이 완료되면 컨테이너는 종료된다.
- 즉, 컨테이너는 프로세스가 있는 동안에만 수명이 있다. 즉, 프로세스가 살아있는 동안에만 살아있다.
- 만약 컨테이너 내부 웹 서비스가 중지거나 충돌된 경우 컨테이너는 종료된다.

**❓ 그렇다면 컨테이너 내부에서 누가 어떤 프로세스가 실행되는지 정의해보자**

- Docker 이미지에 대한 Docker 파일을 보면, CMD 명령이 표시된다.

```yaml
CMD ["nginx"]
```

- 시작할 때, 컨테이너 내에서 실행될 프로그램을 정의하는 명령어이다.
- 따라서, nginx 경우 nginx 명령어를, mysql 이미지의 경우 mysqld 명령어이다.

### Ubuntu

- ubutu 이미지에 대한 도커파일은 bash를 기본 명령어를 사용한다.
- bash는 실제 프로세스가 아니고, 터미널에서 입력을 수신하는 쉘이다.
    - 따라서 터미널을 찾을 수 없으면 종료한다.

- Ubuntu 컨테이너를 실행했을 때, Docker는 Ubuntu 이미지에서 컨테이너를 만들었다.
- bash 프로그램을 시작한다.
- 기본적으로 Docker는 터미널을 연결하지 않기 때문에 bash 프로그램은 터미널을 찾지 못한다.
- 그래서 종료된다.

- 따라서 프로세스가 시작된 이후 컨테이너 생성이 완료되면 컨테이너도 종료된다.

❓ **그렇다면 다른 명령을 어떻게 지정할까?**

- command를 추가한다.
- 그렇게 하면 이미지에서 지정된 기본 명령을 재정의한다.

```yaml
docker run ubuntu [command]

docker run ubuntu sleep 5
```

- 컨테이너가 시작될 때 절전 프로그램을 실행하고 5초 동안 대기한다. 이후 종료한다.

❓ **이 변화를 영구적으로 만드는 방법은 무엇일까?**

- 자신의 이미지를 만든다.
- 기본 우분투 이미지에서 새 명령을 지정한다.

```yaml
CMD command param1 --------------------> CMD sleep 5
CMD ["command", "param1"] -------------> CMD ["sleep", "5"]
```

- 이후 나만의 이미지를 빌드한다.

```yaml
docker build -t ubuntu-sleeper .
docker run ubuntu-sleeper
```

❓ **위 컨테이너의 sleep 시간을 변경하려면 어떻게 해야할까?**

```yaml
docker run ubutun-sleeper sleep 10
```

- ubuntu-sleeper는 그 자체로 컨테이너가 휴먼 상태임을 의미한다.
- 따라서 sleep 커맨드를 다시 지정하지 않기를 원한다.

- 우리는 컨테이너 휴면해야 하는 시간만 전달하고 싶다.
- 그리고 sleep 커맨드가 자동으로 호출되기를 원한다.
- 따라서 entry point instruction을 사용할 수 있다.

---

# Kubernetes Command

- Ubuntu-sleeper 이미지를 사용하여 새 파드를 생성하자.

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: ubuntu-sleeper-pod
spec:
	containers:
	- name: ubuntu-sleeper
		image: ubuntu-sleeper
```

- 해당 파드가 생성되면 지정된 이미지에서 컨테이너를 생성하고 종료되기 전까지 5초동안 sleep한다.
- 만약 컨테이너를 10초 동안 sleep해야 하는 경우에는 어떻게 하는 것이 좋을까?

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: ubuntu-sleeper-pod
spec:
	containers:
	- name: ubuntu-sleeper
		image: ubuntu-sleeper
		args: ["10"]
```

- args 속성을 배열 형식으로 작성하면 된다.
- ubuntu-sleeper 도커파일은 ENTRYPOINT [”sleep”], CMD [”5”]로 설정되어 있다.
    - ENTRYPOINT는 시작 시 실행되는 커맨드고,
    - CMD는 커맨드에 전달되는 default 파라미터이다.
- 따라서 args 옵션을 사용하여 docker 파일의 cmd 커맨드를 재정의한다.

- 그렇다면 엔트리 포인트를 재정의해야하는 경우는?
- Docker 세계에서는 새 커맨드로 설정된 엔트리 포인트 옵션을 사용하여 docker run 커맨들르 실행한다.
- 파드 정의 파일에서는 command 필드를 사용한다.
- 따라서 command는 docker 파일의 엔트리 포인트에 해당한다.

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: ubuntu-sleeper-pod
spec:
	containers:
	- name: ubuntu-sleeper
		image: ubuntu-sleeper
		command: ["sleep2.0"]
		args: ["10"]
```

- docker 파일에는 두 개의 instruction에 해당하는 두 개의 필드가 있다.
    - command 필드는 ENTRY POINT 커맨드를 재정의하고,
    - args 필드는 docker 파일의 명령 커맨드를 재정의한다.
- docker 파일의 cmd 커맨드를 재정의하는 것이 command 필드가 아니다.

---

# 환경변수

```yaml
docker run -e APP_COLOR=pink simple-webapp-color
```

- 파드 정의에서는 env 필드를 사용하면 된다.

```yaml
apiVersion: v1
kind: Pod
metadata: 
	name: simple-webapp-color
spec:
	containers:
	- name: simple-webapp-color
		image: simple-webapp-color
		ports:
		- containerPort: 8080
		env:
		- name: APP_COLOR
			value: pink
```

- env는 배열이므로 env 속성 아래 모든 항목은 배열의 항목으로 나타내는 - 로 시작한다.
- 각 항목에는 이름과 값 속성이 있다.
- 이름은 컨테이너와 함께 사용할 수 있는 환경 변수 이름이고, 값은 해당 값이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c005ca4a-dafb-41d1-ab48-85dede7fd2b4/Untitled.png)

- config map과 secret를 사용하는 것에는 환경 변수를 설정하는 다른 방법이 있다.
- value 대신 valueFrom을 사용하고, config map 혹은 secret의 specification을 입력한다.

---

# ConfigMap

- 파드 정의 파일에서 환경 변수를 정의하는 방법이 있다.
- 만약 파드 정의 파일이 많으면 query’s file에 저장된 환경 데이터를 관리하기 어려워진다.
- 파드 정의 파일에서 정보들을 가져와 config map을 사용하여 중앙에서 관리할 수 있게 한다.

### configmap

- 쿠버네티스에서 키 값 쌍의 형태로 configuration 데이터를 전달하는데 사용된다.
- 파드가 생성되면 config map을 파드에 삽입하여 파드의 컨테이너 내부에서 호스팅되는 애플리케이션의 환경 변수로 키 값 쌍을 사용할 수 있다.
- config map config에는 두 단계가 있다.
    - config map을 만든다.
    - 파드에 주입한다.
- config map을 만드는 방법에도 두가지가 있다.
    - config map definition 파일을 사용하지 않은 커맨드형 방식
    - config map definition 파일을 사용하는 선언적 방식이다.

### command 방식

- kubectl을 사용하여 create configmap 커맨드를 입력하고
- 필요한 arguments를 지정한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b6371668-5dca-4d53-9f00-20c379eaf8e7/Untitled.png)

- 커맨드 라인에 키 값 쌍을 직접 지정한다.
- 주어진 값의 config map을 생성하려면 kubectl create config map 커맨드를 실행한다.
- 커맨드 뒤에는 config 이름과
    
    ```yaml
    --from-literal로 키 값을 여러번 지정하거나
    --from-file로 데이터가 포함된 파일의 경로를 지정한다
    ```
    
    ```yaml
    kubectl create configmap \
    app-cofing --from-literal=APP_COLOR=blue \ 
    					 --from literal=APP_NODE=prod
    ```
    
    ```yaml
    kubectl create configmap \
    app-config --from-file=app_config.properties
    ```
    

### defintion 방식

- definition 파일을 생성한다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
	name: app-config
data:
	APP_COLOR: blue
	APP_NODE: prod
```

```yaml
kubectl create -f config.yaml
```

```yaml
kubectl get configmaps
kubectl describe configmaps
# 데이터까지 볼 수 있다.
```

- 파드에 config를 지정한다.

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
		envFrom:
		- configMapRef:
			name: app-config
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/326ac6f4-90af-4448-88b0-212977da9668/Untitled.png)

---

# Secrets

- configmap은 데이터를 일반 텍스트 형식으로 저장한다.
- 따라서 보안 관점에서 취약하다.
- 암호를 저장하기 적합한 저장소가 필요하다.
- Secrets는 민감한 정보를 저장하는데 사용된다.
- configmap과 유사하지만 암호화된 형식 또는 해시 형식으로 저장된다.

- 명령형 방법

```yaml
kubectl create secret generic \
<secret-name> --from-literal=<key>=<value>

kubectl create secret generic \
<secret-name> --from-file=<path-to-file>
```

- 선언형 방법

```yaml
apiVersion: v1
kind: Secret
metadata:
	name: app-secret
data:
	DB_Host: KDJFAED_
	DB_User: SKFEJF
	DB_Password: dkfejfsK
```

- 인코딩된 형식으로 데이터를 지정한다.
    
    ```yaml
    echo -n 'mysql' | base64
    KDJFAED_
    
    echo -n 'root' | base4
    SKFEJF
    ```
    
- 디코딩
    
    ```yaml
    echo -n 'KDJFAED_' | base64 --decode
    mysql
    ```
    
- view secret
    
    ```yaml
    kubectl get secrets
    
    kubectl get secret app-secret -o yaml
    ```
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/87c75580-3d0b-4e43-9d35-92dc83d5c82b/Untitled.png)
    
- Pod에 Secret 설정
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/16a8b1e2-8aa5-4b3b-afbd-48c80f427c79/Untitled.png)
    
    - 단일 환경변수로 추가될 수 있다.
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/932333e2-e043-4a8b-98e2-f01f54e84e81/Untitled.png)
    
    - 볼륨에 추가 가능하다.
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/646c86fb-5667-48c9-b289-732c158b0d09/Untitled.png)
    

---

- Secrets는 base64 형식으로 데이터를 인코딩한다.
- base64로 인코딩된 secret를 가진 사람은 누구나 쉽게 secret을 해독할 수 있다.
    - 따라서 secret이 그다지 안전하지 않은 것으로 간주될 수 있다.
- secret은 암호화되지 않으므로 안전하지 않다.
- 하지만 secret 사용에 대한 모범 사례는 secret을 더 안전하게 만든다.
    - secret object 정의 파일을 소스 코드 리포에 checking-in X
    - ETCD에 암호화되어 저장되도록 secret에 대한 Encryption at REST 활성화
- 쿠버네티스는 secret을 다음과 같이 처리한다.
    - secret은 해당 노드의 파드에 필요한 경우에만 노드로 전송된다.
    - kubelet은 secret이 디스크 저장소에 기록되지 않도록 secret을 tmpfs에 저장
    - secret에 의존하는 파드가 삭제되면 kubelet은 secret 데이터의 로컬 복사본도 복제한다.
