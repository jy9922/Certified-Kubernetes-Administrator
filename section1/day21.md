### Application Failure

- curl을 사용하여 웹 서버가 노드 포트 IP에 접근 가능한지 확인한다.

```yaml
curl http://web-service-ip:node-port
```

- 연결이 안된다면, 서비스를 체크한다.

```yaml
kubectl describe service web-service
```

- 해당 명령어를 통해 서비스에 구성된 selector와 파드에 있는 항목을 비교하여 일치하는지 확인한다.
- 파드가 Running 상태인지 확인한다.
- 파드 관련 이벤트를 본다.
- 애플리케이션 로그를 확인한다.
    - 로그의 경우, 파드가 재시작하면 이전의 로그는 볼 수 없다. 따라서 -f 옵션으로 관찰하거나 —previous 옵션으로 이전 파드의 로그를 본다.

```yaml
kubectl logs web -f --previous
```

---

Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.

**Troubleshooting Test 1:** A simple 2 tier application is deployed in the `alpha` namespace. It must display a green web page on success. Click on the `App` tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

```yaml
controlplane ~ ✖ k get svc -n alpha mysql -o yaml > svc.yaml

controlplane ~ ➜  k delete -n alpha svc mysql 
service "mysql" deleted

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2023-03-13T15:37:01Z"
  **name: mysql-service**
  namespace: alpha

controlplane ~ ➜  k apply -f svc.yaml
```

**Troubleshooting Test 2:** The same 2 tier application is deployed in the `beta` namespace. It must display a green web page on success. Click on the `App` tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.

```yaml
ports:
  - port: 3306
    protocol: TCP
    targetPort: 3306
```

**Troubleshooting Test 3:** The same 2 tier application is deployed in the `gamma` namespace. It must display a green web page on success. Click on the `App` tab at the top of your terminal to view your application. It is currently failed or unresponsive. Troubleshoot and fix the issue.

Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.

```yaml
**Selector:          name=sql00001**
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.43.243.200
IPs:               10.43.243.200
Port:              <unset>  3306/TCP
TargetPort:        3306/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>

controlplane ~ ➜  k describe -n gamma po mysql 
Name:             mysql
Namespace:        gamma
Priority:         0
Service Account:  default
Node:             controlplane/172.25.0.83
Start Time:       Mon, 13 Mar 2023 16:17:04 +0000
**Labels:           name=mysql**
```

**roubleshooting Test 4:** The same 2 tier application is deployed in the `delta` namespace. It must display a green web page on success. Click on the `App` tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.

```yaml
- env:
    - name: DB_Host
      value: mysql-service
    - name: DB_User
      **value: root**
    - name: DB_Password
      value: paswrd
```

**Troubleshooting Test 5:** The same 2 tier application is deployed in the `epsilon` namespace. It must display a green web page on success. Click on the `App` tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.
