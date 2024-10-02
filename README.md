# k8s-external-communication

## 도입 이유
 쿠버네티스는 도커와 같은 컨테이너 기술과 다양한 클라우드 환경(AWS, GCP, Azure 등)에서 실행될 수 있어 이러한 특성 때문에 쿠버네티스는 현대의 마이크로서비스 아키텍처를 지원하는 데 있어 필수적인 도구로 자리잡고 있어 기업에서 적극활용하여 해당 기술을 공부하고자 구현을 해보았습니다.

## 소개

쿠버네티스는 컨테이너화된 애플리케이션의 배포, 확장 및 관리를 자동화하기 위한 오픈 소스 플랫폼으로 클라우드 환경에서 애플리케이션을 더 효율적으로 운영할 수 있도록 설계되었습니다.
 
 쿠버네티스는 복잡한 컨테이너 관리 작업을 추상화하고 자동화함으로써, 개발자와 운영자가 인프라에 대한 지식의 부재를 해결해줍니다.
 
 쿠버네티스는 컨테이너 오케스트레이션 도구로서, 여러 컨테이너의 배치, 스케일링 및 네트워킹을 관리합니다. 이를 통해 고가용성, 확장성, 장애복구와 같은 클라우드 네이티브 애플리케이션의 요구사항을 충족시킵니다.
 
## 구현
다음 아키텍처의 형상을 구현하였습니다.
![image](https://github.com/user-attachments/assets/d90322a4-318b-477d-bcde-a72290554be2)


### Kubernetes yaml 파일 생성
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 3  # 3개의 Pod 생성
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: nginx  # Nginx
        ports:
        - containerPort: 80  # 컨테이너 내부 포트

---
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app  # Service가 연결될 Pod들을 선택하는 라벨
  type: LoadBalancer  # 외부 통신을 위해 LoadBalancer 타입 설정
  ports:
  - protocol: TCP
    port: 80        # 외부에서 접근할 포트
    targetPort: 80  # Pod 내부에서 서비스되는 포트 (컨테이너의 containerPort와 일치)
    nodePort: 30007
```

### 서비스 정상 실행

```bash
kubectl apply -f "yml파일명"
```

```bash
kubectl get services or svc

NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/kubernetes       ClusterIP      10.96.0.1        <none>        443/TCP        44m
service/my-app-service   LoadBalancer   10.100.160.195   <pending>     80:30007/TCP   17m
```

### 트러블 슈팅

일반적인 생성만으로 외부에서 접근하려하였을때 EXTERNAL-IP가 <pending>상태로 되는 것을 보았습니다.

minikube tunnel을 사용할 경우 가상 외부 IP가 할당됩니다. 
해당 명령어를 통해 외부와의 통신이 가능하여 사용하였습니다.

```bash
username@servername:~$ minikube tunnel
Status:
        machine: minikube
        pid: 414913
        route: 10.96.0.0/12 -> 할당된 IP
        minikube: Running
        services: [my-app-service]
    errors: 
                minikube: no errors
                router: no errors
                loadbalancer emulator: no errors
```

```bash
kubectl get service

NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)        AGE
service/kubernetes       ClusterIP      10.96.0.1        <none>           443/TCP        28m
service/my-app-service   LoadBalancer   10.100.160.195   10.100.160.195   80:30007/TCP   37s
```
외부 접속 IP인 10.100.160.195가 <pending>상태에서 변경되었습니다.

### 실행 결과
EXTERNAL-IP와 nodePort를 결합하여 접속을 하였습니다.
![image](https://github.com/user-attachments/assets/7750d662-e6ae-4eef-8960-aa92b595d22b)

작업이 성공되었습니다.
