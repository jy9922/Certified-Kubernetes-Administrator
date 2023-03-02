# Service Account

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e5f28104-bcb9-472e-a6b2-4f4a7c121f9e/Untitled.png)

- 쿠버네티스는 서비스 계정과 사용자 계정으로 나뉘어져 있다.
    - 사용자 계정은 관리 작업을 수행하기 위해 클러스터에 액세스하는 관리자 또는 개발자 등을 위한 계정이다.
    - 서비스 계정은 쿠버네티스 클러스터와 상호작용하기 위해 애플리케이션에서 사용하는 계정이다. (ex. Prometheus, Jenkins)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2c169dd7-bb58-494b-94e2-75408a1d40e3/Untitled.png)

- 애플리케이션이 **Kubernetes API에 쿼리하려면** 인증을 받아야한다. 이를 위해 서비스 계정을 이용한다.

- 서비스 계정을 생성해보자.

```yaml
kubectl create serviceaccount dashboard-sa
```

- 서비스 계정 리스트를 보자.

```yaml
kubectl get serviceaccount
```

- 서비스 계정의 내용을 자세히 보자.

```yaml
kubectl describe serviceaccount dashboard-sa
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/93bf4ae1-2966-4a21-b9bf-1a7572851886/Untitled.png)

- 서비스 계정이 생성되면 토큰도 자동 생성된다.
    - 서비스 계정 토큰은 쿠버네티스 API에 인증하기 위해 외부 애플리케이션에서 사용하는 것이다.
- 토큰은 secret 오브젝트에 저장된다.
    - 먼저, 서비스 계정을 만든 후 서비스 계정에 대한 토큰을 생성한다. 이후 secret 오브젝트를 만들고, secret 오브젝트에 해당 토큰을 저장한다.
    - 그리고 해당 secret 오브젝트가 서비스 계정에 연결된다.
- 토큰을 보려면 secret 오브젝트를 보면된다.

```yaml
kubectl describe secret 토큰이름
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2edf9109-b8b8-4187-8bfa-1629dd018dae/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/53a1bba7-95cb-4157-9fc8-5b32288f93c2/Untitled.png)

- 해당 토큰은 쿠버네티스 API에 대한 risk call을 수행하는 동안 bearer token을 인증 헤더로 제공할 수 있다.
- 따라서 해당 애플리케이션에서는 토큰을 복사하여 토큰 필드에 붙여 인증한다.
- 이렇게 서비스 계정을 생성하고, 토큰을 내보내서 외부 애플리케이션이 쿠버네티스 API에 인증할 수 있게되는 것이다.

**만약, 외부 애플리케이션이 쿠버네티스 클러스터 자체에서 호스팅된다면 어떻게 할까?**

- **서비스 토큰 암호를 타 애플리케이션을 호스팅하는 파드 내부 볼륨으로 자동 마운트**하도록 하면 된다.
- 이렇게 하면 쿠버네티스 API에 액세스하기 위한 토큰이 이미 파드 내부에 배치되어 있기 때문에 애플리케이션에서 수동으로 제공할 필요가 없어지게 된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b42af9d6-8ffb-4a7b-ac55-79d8e784af0b/Untitled.png)

- 쿠버네티스의 모든 namespace에 대해 **default 서비스 계정**이 있다.
- 파드가 생성될 때마다 default 서비스 계정과 해당 토큰이 볼륨 마운트로 해당 파드에서 자동으로 마운트된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3fea5197-3141-45e8-97ad-7d579afe1b13/Untitled.png)

- 애플리케이션을 호스팅하기 위해 파드 생성을 한 후, 자세히 보기로 확인해보면 서비스 계정과 토큰이 자동으로 생성된 것을 확인할 수 있다.
- secret 토큰은 파드 내부의 /var/run/secrets/kubernetes.io/serviceaccount 위치에 마운트된다.
- 따라서 파드를 실행 후 확인해보면 토큰이 있는 것을 확인할 수 있다.

```yaml
kubectl exec -it my-kubernetes-dashboard ls /var/run/kubernetes.io/serviceaccount
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2d323570-7cff-4de5-846e-0f55819f37cc/Untitled.png)

**default 서비스 계정은 매우 제한적이다**

- 쿠버네티스 API 쿼리를 실행할 수 있는 권한만 있다.
- 다른 서비스 계정을 사용하려면 서비스 계정을 포함하도록 definition 파일을 수정하고 서비스 계정 이름을 생성해야 한다.
- 기존에 있는 서비스 계정은 edit 할 수 없기 때문에 기존 파드를 삭제 후 다시 생성해야 한다.
- deployment 경우는 파드 definition 파일을 생성하면 배포에 대한 새 롤아웃이 자동 트리거되기 때문에 서비스 계정을 사용할 수 있다. deployment가 알아서 기존 파드를 삭제하고 새 파드 다시 생성해주는 작업을 처리한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e7dced21-8859-4cd6-ac12-a9cda31b5177/Untitled.png)

- 명시적으로 서비스 계정을 지정하지 않은 경우, 쿠버네티스는 자동으로 default 서비스 계정을 마운트한다.
- 따라서 spec 섹션에 automountServiceAccountToke: false를 통해 서비스 계정에 대한 자동 마운트하지 않도록 할 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/229b581c-444e-4c54-a31e-8a798f389d0a/Untitled.png)

**정리**

- 모든 namespace에는 default serviceaccount가 있다.
- 그리고 해당 서비스 계정에 연결된 토크이 있는 secret 오브젝트가 있다.
- 파드가 생성되면 자동으로 서비스 계정 파드에 연결하고, 토큰을 파드 내에서 잘 알려진 위치에 마운트한다. (/var/run/secrets/kubernetes.io/serviceaccount
- 파드 내에 있는 프로세스가 쿠버네티스 API 쿼리를 할 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/605dec56-4326-4ee2-ad67-d448790c554e/Untitled.png)

- 토큰은 해당 경로로 들어가 볼 수 있다.
- 해당 토큰을 가지고 토큰에 대한 디코딩도 가능하다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/45c711a5-4aa8-42bc-8be7-04d20b0d79f5/Untitled.png)

**버전에 따른 토큰 관리**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fcd987d4-8a44-4f29-a931-e59a89fc1f6f/Untitled.png)

- 과거
    - 서비스 계정 생성될 때 만료가 없고 대상에 구속이 없는 토큰으로 secret 오브젝트 자동 생성
- v1.22
    - 파드에 대한 secret 오브젝트의 자동 마운트가 변경되고, 토큰 요청 API로 변경됨
- v1.24
    - 서비스 계정 생성할 때마다, secret 오브젝트 또는 토큰을 secret으로 자동 생성하지 않도록 변경됨

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5b2b5580-fdf7-462d-a8eb-09997f35ce3e/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f17f81b5-4d9f-4c12-bb54-e244e657af97/Untitled.png)

- 해당 토큰을 디코딩하면 만료 날짜가 정의되어있음을 확인할 수 있다.
- 시간 제한을 지정하지 않은 경우, 일반적으로 커맨드 실행 시간부터 1시간이다.
- 컨맨드 추가 옵션으로 토큰 만료 날짜를 늘릴 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bda65be3-1307-4ea1-ab3c-1188ace82150/Untitled.png)

- 만료되지 않은 토큰을 사용하고자하면, service 계정을 생성 후 secret을 다음과 같이 생성한다.
- annotations을 이용하여 해당 토큰을 서비스 계정에 연결시킬 수 있다.

하지만! 서비스 계정 토큰은 secret 오브젝트를 사용하는 대시 토큰 API를 사용하는 것이 좋다. 왜? 생명주기가 제한되고 더 안전하기 때문이다.

# Image Security

### Image

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a1c3c4e7-5767-4cb7-b58c-88f046927f5c/Untitled.png)

- Docker 이미지 명명 규칙을 따른다.
- niginx → library/nginx
    - 첫번째 부분은 사용자 또는 계정 이름을 나타낸다.
    - 사용자 또는 계정 이름을 제공하지 않으면 라이브러리로 간주된다. **library는 Docker의 공식 이미지가 저장되는 default 계정 이름**이다.
- 해당 이미지는 어디에서 저장되고 어디에서 가져올까?
    - 이미지를 가져올 위치를 지정하지 않으면, Docker의 default 레지스트리인 Docker Hub로 가져온다.
    - DNS 이름은 [docker.io](http://docker.io) 이다.
    - 레지스트리는 모든 이미지가 저장되는 곳이다.
    - Google 레지스트리 → [gcr.io](http://gcr.io) (public)
    - public이 아닌 private으로 하고 싶은 경우, private 레지스트리를 호스팅하는 것이 좋다.
        
        (AWS, Azure, GCP 등 클라우드 서비스는 private 레지스트리를 제공한다.)
        

### Private Registry

- 자격 증명 세트를 사용해, 액세스할 수 있도록 리포지토리를 비공개로 만들 수 있다.
- Docker 관점에서 private 레지스트리를 실행하려면 먼저 docker 로그인을 통해 private 레지스트리에 로그인한다.

```yaml
docker login private-registry.io
로그인
```

- 파드 구성 파일에서 이미지 이름을 수정한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6586c7dd-77eb-404c-8a0f-11383c7437b6/Untitled.png)

- 이미지를 Pull 하기 위한 자격 증명을 위해 secret을 만든다.

# Security Context

- 쿠버네티스의 파드 수준에서 보안 설정을 구성하면 파드 내 모든 컨테이너로 전달된다.
- 파드와 컨테이너 모두에서 구성하는 경우, 컨테이너의 설정이 파드 설정을 오버라이드한다.

- 사용자로 실행 옵션을 사용하여 파드의 사용자 ID를 설정한다.

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: web-pod
spec:
	containers:
	- name: ubuntu
		image: ubuntu
		command: ["sleep", "3600"]
		securityContext:
			runAsUser: 1000
```

- 이후 capability를 추가한다.

