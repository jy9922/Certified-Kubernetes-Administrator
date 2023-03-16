Take a backup of the etcd cluster and save it to `/opt/etcd-backup.db`.

```yaml
controlplane ~ ➜  ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379   --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \> snapshot save /opt/etcd-backup.db
Snapshot saved at /opt/etcd-backup.db
```

Create a Pod called `redis-storage` with image: `redis:alpine` with a Volume of type `emptyDir` that lasts for the life of the Pod.

Specs on the below.

- Pod named 'redis-storage' created
- Pod 'redis-storage' uses Volume type of emptyDir
- Pod 'redis-storage' uses volumeMount with mountPath = /data/redis

```yaml
controlplane ~ ➜  k run redis-storage --image redis:alpine --dry-run=client -o yaml > pod.yaml

controlplane ~ ➜  vi pod.yaml 

controlplane ~ ➜  k apply -f pod.yaml 
pod/redis-storage created

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis-storage
  name: redis-storage
spec:
  containers:
  - image: redis:alpine
    name: redis-storage
    resources: {}
    volumeMounts:
    - mountPath: /data/redis
      name: redis-volume
  volumes:
  - name: redis-volume
    emptyDir: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Create a new pod called `super-user-pod` with image `busybox:1.28`. Allow the pod to be able to set `system_time`.

The container should sleep for 4800 seconds.

- Pod: super-user-pod
- Container Image: busybox:1.28
- SYS_TIME capabilities for the conatiner?

```yaml
controlplane ~ ➜  k run super-user-pod --image busybox:1.28 --dry-run=client -o yaml > pod2.yaml

controlplane ~ ➜  vi pod2.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: super-user-pod
  name: super-user-pod
spec:
  containers:
  - image: busybox:1.28
    name: super-user-pod
    resources: {}
    command: ["sleep", "4800"]
		securityContext:
      capabilities:
        add: ["SYS_TIME"]
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

controlplane ~ ➜  k apply -f pod2.yaml 
pod/super-user-pod created
```

[https://kubernetes.io/docs/tasks/configure-pod-container/security-context/](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

A pod definition file is created at `/root/CKA/use-pv.yaml`. Make use of this manifest file and mount the persistent volume called `pv-1`. Ensure the pod is running and the PV is bound.

mountPath: `/data`persistentVolumeClaim Name: `my-pvc`

- persistentVolume Claim configured correctly
- pod using the correct mountPath
- pod using the persistent volume claim?

```yaml
controlplane ~/CKA ➜  k describe persistentvolume pv-1
Name:            pv-1
Labels:          <none>
Annotations:     <none>
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    
Status:          Available
Claim:           
Reclaim Policy:  Retain
**Access Modes:    RWO**
VolumeMode:      Filesystem
**Capacity:        10Mi**
Node Affinity:   <none>
Message:         
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /opt/data
    HostPathType:  
Events:            <none>

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Mi

controlplane ~/CKA ➜  k apply -f pvc.yaml 
persistentvolumeclaim/my-pvc created
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: use-pv
  name: use-pv
spec:
  **volumes:
    - name: my-pvc
      persistentVolumeClaim:
        claimName: my-pvc**
  containers:
  - image: nginx
    name: use-pv
    resources: {}
    **volumeMounts:
      - mountPath: "/data"
        name: my-pvc**
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

controlplane ~/CKA ➜  k apply -f use-pv.yaml 
pod/use-pv created

controlplane ~/CKA ➜  k get po,pv,pvc
NAME                 READY   STATUS    RESTARTS   AGE
pod/redis-storage    1/1     Running   0          17m
pod/super-user-pod   1/1     Running   0          13m
pod/use-pv           1/1     Running   0          11s

NAME                    CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM            STORAGECLASS   REASON   AGE
persistentvolume/pv-1   10Mi       RWO            Retain           Bound    default/my-pvc                           13m

NAME                           STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/my-pvc   Bound    pv-1     10Mi       RWO                           2m48s
```

Create a new deployment called `nginx-deploy`, with image `nginx:1.16` and `1` replica. Next upgrade the deployment to version `1.17` using rolling update.

- Deployment : nginx-deploy. Image: nginx:1.16
- Image: nginx:1.16
- Task: Upgrade the version of the deployment to 1:17
- Task: Record the changes for the image upgrade

```yaml
controlplane ~/CKA ➜  k create deployment nginx-deploy --image nginx:1.16 --replicas 1
deployment.apps/nginx-deploy created

controlplane ~/CKA ➜  kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/nginx-deploy image updated
```

🔥 **Create a new user called `john`. Grant him access to the cluster. John should have permission to `create, list, get, update and delete pods` in the `development` namespace . The private key exists in the location: `/root/CKA/john.key` and csr at `/root/CKA/john.csr`.**

`**Important Note`: As of kubernetes 1.19, the CertificateSigningRequest object expects a `signerName`.Please refer the documentation to see an example. The documentation tab is available at the top right of terminal.**

- **CSR: john-developer Status:Approved**
- **Role Name: developer, namespace: development, Resource: Pods**
- **Access: User 'john' has appropriate permissions**

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john-developer
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZVzVuWld4aE1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQTByczhJTHRHdTYxakx2dHhWTTJSVlRWMDNHWlJTWWw0dWluVWo4RElaWjBOCnR2MUZtRVFSd3VoaUZsOFEzcWl0Qm0wMUFSMkNJVXBGd2ZzSjZ4MXF3ckJzVkhZbGlBNVhwRVpZM3ExcGswSDQKM3Z3aGJlK1o2MVNrVHF5SVBYUUwrTWM5T1Nsbm0xb0R2N0NtSkZNMUlMRVI3QTVGZnZKOEdFRjJ6dHBoaUlFMwpub1dtdHNZb3JuT2wzc2lHQ2ZGZzR4Zmd4eW8ybmlneFNVekl1bXNnVm9PM2ttT0x1RVF6cXpkakJ3TFJXbWlECklmMXBMWnoyalVnald4UkhCM1gyWnVVV1d1T09PZnpXM01LaE8ybHEvZi9DdS8wYk83c0x0MCt3U2ZMSU91TFcKcW90blZtRmxMMytqTy82WDNDKzBERHk5aUtwbXJjVDBnWGZLemE1dHJRSURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBR05WdmVIOGR4ZzNvK21VeVRkbmFjVmQ1N24zSkExdnZEU1JWREkyQTZ1eXN3ZFp1L1BVCkkwZXpZWFV0RVNnSk1IRmQycVVNMjNuNVJsSXJ3R0xuUXFISUh5VStWWHhsdnZsRnpNOVpEWllSTmU3QlJvYXgKQVlEdUI5STZXT3FYbkFvczFqRmxNUG5NbFpqdU5kSGxpT1BjTU1oNndLaTZzZFhpVStHYTJ2RUVLY01jSVUyRgpvU2djUWdMYTk0aEpacGk3ZnNMdm1OQUxoT045UHdNMGM1dVJVejV4T0dGMUtCbWRSeEgvbUNOS2JKYjFRQm1HCkkwYitEUEdaTktXTU0xMzhIQXdoV0tkNjVoVHdYOWl4V3ZHMkh4TG1WQzg0L1BHT0tWQW9FNkpsYWFHdTlQVmkKdjlOSjVaZlZrcXdCd0hKbzZXdk9xVlA3SVFjZmg3d0drWm89Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth

controlplane ~/CKA ➜  kubectl create role developer --verb=create,list,get,update,delete --resource=pod -n development 
role.rbac.authorization.k8s.io/developer created
```

```yaml
kubectl certificate approve john-developer
```

```yaml
$ kubectl create role developer --resource=pods --verb=create,list,get,update,delete --namespace=development

$ kubectl create rolebinding developer-role-binding --role=developer --user=john --namespace=development
```

```yaml
kubectl auth can-i update pods --as=john --namespace=development
```

Create a nginx pod called `nginx-resolver` using image `nginx`, expose it internally with a service called `nginx-resolver-service`. Test that you are able to look up the service and pod names from within the cluster. Use the image: `busybox:1.28` for dns lookup. Record results in `/root/CKA/nginx.svc` and `/root/CKA/nginx.pod`

- Pod: nginx-resolver created
- Service DNS Resolution recorded correctly
- Pod DNS resolution recorded correctly

```yaml
controlplane ~/CKA ➜  k run nginx-resolver --image nginx 
pod/nginx-resolver created

controlplane ~/CKA ➜  kubectl expose pod nginx-resolver --port=80 --target-port=80 --name nginx-resolver-service
service/nginx-resolver-service exposed

controlplane ~/CKA ➜  k apply -f dns-pod.yaml 
pod/dnsutils created

controlplane ~/CKA ➜  kubectl exec -i -t dnsutils -- nslookup nginx-resolver-service
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   nginx-resolver-service.default.svc.cluster.local
Address: 10.98.17.109

controlplane ~/CKA ➜  kubectl exec -i -t dnsutils -- nslookup nginx-resolver-service > /root/CKA/nginx.svc
```

Create a static pod on `node01` called `nginx-critical` with image `nginx` and make sure that it is recreated/restarted automatically in case of a failure.

Use `/etc/kubernetes/manifests` as the Static Pod path for example.

- static pod configured under /etc/kubernetes/manifests ?
- Pod nginx-critical-node01 is up and running

```yaml
controlplane ~/CKA via 🐪 v5.30.0 ➜  ssh node01

root@node01 /etc/kubernetes/manifests ➜  vi nginx-pod.yaml

controlplane ~/CKA via 🐪 v5.30.0 ➜  k get po -o wide 
NAME                           READY   STATUS    RESTARTS   AGE     IP             NODE     NOMINATED NODE   READINESS GATES
dnsutils                       1/1     Running   0          4m40s   10.244.192.6   node01   <none>           <none>
nginx-critical-node01          1/1     Running   0          21s     10.244.192.7   node01   <none>           <none>

root@node01 /etc/kubernetes/manifests ➜  cat nginx-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: nginx-critical
  labels:
    role: myrole
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          protocol: TCP
```
