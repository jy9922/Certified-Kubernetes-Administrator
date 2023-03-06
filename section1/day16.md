# Persistent Volume & Persistent Volume Claim

### Persistent Volume

- 관리자가 중앙에서 관리할 수 있는 대규모 스토리지 풀을 생성한 다음 사용자가 필요에 따라 풀을 분할할 수 있도록 구성할 수 있다.
- 바로 Persistent Volume을 통해!

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e9c77e66-8aae-437a-b729-0aada15edfce/Untitled.png)

- Persistent volume은 클러스터에 애플리케이션을 배포하는 사용자가 사용하도록 관리자가 구성한 클러스터 범위의 스토리지 볼륨 풀이다.
- 사용자는 persistent volume claims를 통해 풀에서 스토리지를 선택할 수 있다.

```yaml
kind: PersitentVolume
apiVersion: v1
metadata: 
	name: pv-vol1
spec:
	accessModes: ["ReadWriteOnce"]
	capacity: 
		storage: 1Gi
	hostPath:
		path: /tmp/data
```

- spec 섹션에는 accessModes가 있다.
    - 액세스 모드는 읽기 전용 모드인지 읽기/쓰기 전용 모드인지 호스트에 볼륨을 마운트하는 방법을 정의한다.
    - ReadOnlyMany, ReadWriteOnce, ReadWriteMany
- storage에는 persistent 볼륨에 대해 스케줄링할 스토리지 양을 지정한다.
- 볼륨 유형은 hostPath로 노드의 로컬 디렉토리 저장소를 사용한다.

```yaml
k create -f pv-definition.yaml

k get pv
k delete pv pv-vol1
```

---

### Persistent Volume Claim

- 노드에서 스토리지를 사용할 수 있도록 Persistent Volume Claims를 생성하려고 한다.
- Persistent Volume 및 Persistent Volume Claims는 쿠버네티스 네임스페이스에서 개별 오브젝트이다.
- 관리자는 PV 세트를 생성하고, 사용자는 스토리지 사용을 위한 PVC를 생성한다.
- PVC가 생성되면, 쿠버네티스는 request와 volume에 설정된 속성을 기반으로 PV를 PVC에 바인딩한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ebb2fe5a-ee00-4ff1-a70c-b02c9e611d6f/Untitled.png)

- 모든 PVC는 단일 PV에 바인딩된다.
- 바인딩 프로세스 중에 쿠버네티스는 claims에서 요청한대로 용량이 충분하 PV를 찾으려고 한다.
- 액세스 모드, 볼륨 모드, 스토리지 클래스와 같은 기타 request 속성도 반영된다.

- 만약 가능한 볼륨이 여러개 있으며,  특정 볼륨을 사용할 수 있다.
    - label과 selector를 이용하면 된다.
- 하지만, 경우에 따라 작은 claim이 큰 볼륨에 바인딩될 수 있다.
    - claim과 볼륨은 일대일 관계이기 때문에 claim은 볼륨의 나머지 용량을 사용할 수 없다.
- 사용 가능한 볼륨이 없는 경우, PVC는 클러스터에서 새 볼륨을 찾을 수 있을 때까지 Pending 상태로 유지된다.
- 만약 최신 볼륨을 사용할 수 있게 되면 claim은 자동으로 새로 사용 가능한 볼륨에 바인딩된다.

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
	name: myclaim
spec:
	accessModes: ["ReadWriteOnce"]
	resources:
		requests:
			storage: 500Mi
```

```yaml
k create -f pvc.yaml
```

- PVC를 삭제하면 PV는 어떻게 할지 정할 수 있다. (persistentVolumeReclaimPolicy)
    - default : 볼륨을 유지하는 것이다.
    - Retain : 관리자가 수동으로 삭제할 때까지 데이터가 남아 있음을 의미한다. (다른 claim에서 재사용 불가)
    - Delete : claims 삭제시 volume도 즉시 삭제됨
    - Recycle : PV 내부 데이터는 삭제되지만, 다른 claim이 해당 공간 사용 가능하게

### Using PVCs in PODs
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```
---
# Storage Class

### Static Provisioning Volume

- 애플리케이션에 스토리지가 필요할 때마다 먼저 Google Cloud에서 디스크를 수동으로 프로비저닝한 후 디스크와 동일한 이름을 사용하여 Persistent Volume 정의 파일을 수동으로 생성해야 한다.
- 이를 static provisioning volume이라고 한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ab531e2a-7282-4af1-b800-7f65a24c0019/Untitled.png)

### Dynamic Provisioning

- 볼륨이 자동으로 프로비저닝되는 방법이다.
- 스토리지 클래스가 사용된다.
- 스토리지 클래스는 Google Cloud에서 스토리지를 자동 프로비저닝하고 claims가 있을 때, 파드에 연결할 수 있는 프로비저너를 정의할 수 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e385c3f2-27f5-4c51-b6f9-d92ad858ed06/Untitled.png)

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
	name: google-storage
provisioner: kubernetes.io/gce-pd
```

- PVC를 사용하는 파드가 있고, PVC에는 PV 대신 스토리지 클래스가 있다.
- 스토리지는 스토리지 클래스가 생성될 때 자동 생성된다.
- PVC가 생성되면, 스토리지 클래스는 정의된 프로비저너를 활용해 GCP에서 필요한 크기의 디스크를 프로비저닝한 후 PV를 생성해, PVC에 해당 볼륨을 바인딩한다.
- 수동으로 PV를 생성할 필요가 없는 것! 여전히 PV는 사용된다.

- 프로비저너에는 GCP, AWS EBS, Azure File, Azure Disk 등 다양한 종류가 있다.
- 각 프로비저너에는 다양한 파라미터를 전달할 수 있다.
- 또한 다양한 유형의 디스크를 사용하는 스토리지 클래스를 생성할 수 있다.
