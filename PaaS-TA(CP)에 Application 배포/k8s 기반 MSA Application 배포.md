# PaaS-TA에 MSA 배포
## 1. Architecture 및 배포 순서 설명

1. Architecture 구성도

![Alt text](image.png)

2. Port 번호

![Alt text](image-1.png)

## 2. DB 환경 구축

1. YAML 파일 생성  

`mysql-msa-[board, comment, user].yaml`
```
apiVersion: apps/v1
kind: Deployment
metadata:
    name: mysql-msa-board
spec:
    selector:
        matchLabels:
            app: mysql-board
    strategy:   #pod 교체 전략. Recreate는 기존 pod 모두 삭제 후 다음 새로운 pod 생성
        type: Recreate
    template:
        metadata:
            labels:
                app: mysql-board
        spec:
            containers:
                - image: mysql:5.6
                  name: mysql-board
                  env:
                    - name: MYSQL_ROOT_PASSWORD
                      value: password
                  ports:
                    - containerPort: 3306
                      name: mysql-board
            volumes:
                - name: mysql-board
---
apiVersion: v1
kind: Service
metadata:
    name: mysql-msa-board
spec:
    type: NodePort
    ports:
      - port: 3306
        protocol: TCP
        targetPort: 3306
        nodePort: ${MYSQL_MSA_BOARD}    #외부 포트. ${MYSQL_MSA_BOARD}: 305xx
    selector:
        app: mysql-board
```

2. DB 배포

- DB 배포: `kubectl apply -f ${MYSQL_YAML_NAME}.yaml`

- DB 조회 : `kubectl get pods`

3. container 실행

- pod 내 container 실행 : `kubectl exec -it [POD NAME] /bin/bash` 

- `터미널에서 DB 접속`

    - `mysql -u root -p` 후 비밀번호 입력란에 `password` 입력 
        - 비밀번호는 YAML 파일에서 지정
        - 비밀번호가 맞으면 DB 접속

    - `데이터베이스 생성` : `CREATE DATABASE msa_board default CHARACTER SET UTF8;`
        - `CREATE DATABASE [DATABASE NAME] default CHARACTER SET UTF8;`

    - DB 종료: `exit` 

5. 데이터베이스 연결

- datagrip 이용

- 새 데이터베이스 열기
- MySQL DB 선택
- Server Host에는 ${KUBERNETES_URL}, Database에는 ${DATABASE_NAME} Port에 NodePort, Username에 root, Password에 password 입력
    - ${KUBERNETES_URL} : 쿠버네티스 URL
    - ${MYSQL_MSA_BOARD} : `kubectl get services`의 PORT(S)의 " : " 뒤의 숫자가 NodePort
    - ${Username} : root. yaml 파일
    - ${Password} : password. yaml 파일
- Test Connection
- (Drive 설치 창 출력 시) Download 버튼 클릭해 Driver 설치

6. SQL 스크립트 불러오기 & 실행

- SQL 아이콘 클릭 > 파일 > SQL 스크립트 불러오기

- 스크립트를 불러온 후 스크립트를 실행할 DB 선택 후 오른쪽 마우스 클릭 > 실행 > SQL 스크립트 실행

## 3. 소스 코드 환경 설정 및 war 파일 생성

- msa 서비스의 `properties 파일` 수정

- properties 경로는 src/main/resources/properties/[FILE NAME]

![Alt text](image-2.png)

- 내 서비스의 경우 config 파일로 나뉘어 git에 별도로 올라가있기에 해당 단계 할 필요없이 git의 코드 수정하면 됨

## 4. Docker Image 


## 5. Kubernetes 배포 - YAML

1. `redis-msa-ui.yaml`
```
apiVersion: v1
kind: ConfigMap
metadata:
    name: redis-msa-config
data:
    redis-config: "requirepass password"    #저장할 [key]:[value]
---
apiVersion: apps/v1
kind: Deployment
metadata:
    name: redis-msa-ui
    labels:
        app: redis
spec:
    replicas: 1
    selector:
        matchLabels:
            app: redis
    template:
        metadata:
            labels:
                app: redis
        spec:
            containers:
                - name: redis
                  image: redis:latest
                  command:  #배포시 실행할 명령어
                    - redis-server
                    - "/redis-master/redis.conf"
                  env:      #환경 변수 설정
                    - name: MASTER
                      value: "true"
                  ports:
                    - containerPort: 6379
                  resources:    #리소스 제한 설정
                    limits:
                        cpu: "0.1"
                  volumeMounts: #컨테이너에 볼륨마운트할 위치 설정
                    - mountPath: /redis-master-data
                      name: data
                    - mountPath: /redis-master
                      name: config
            volumes:    #pod에 제공할 볼륨 지정
              - name: data
                emptyDir: {}    #emptyDir: pod 생성시 생성되는 임시 볼륨
              - name: config
                configMap:
                    name: redis-msa-config  #mount하려는 configMap의 이름 지정
                    items:  #configMap에서 파일로 생성할 key 배열
                        - key: redis-config
                          path: redis.conf
---
apiVersion: v1
kind: Service
metadata:
    name: redis-msa-ui
    labels:
        app: redis
spec:
    ports:
        - nodePort: ${REDIS_MSA_UI} #외부 port. 308xx
          port: 6379
          protocol: TCP
          targetPort: 6379
    selector:
        app: redis
    type: NodePort
```

2. `mysql-msa-[board, comment, user].yaml`
```
apiVersion: apps/v1
kind: Deployment
metadata:
    name: mysql-msa-board
spec:
    selector:
        matchLabels:
            app: mysql-board
    strategy:   #pod 교체 전략. Recreate는 기존 pod 모두 삭제 후 다음 새로운 pod 생성
        type: Recreate
    template:
        metadata:
            labels:
                app: mysql-board
        spec:
            containers:
                - image: mysql:5.6
                  name: mysql-board
                  env:
                    - name: MYSQL_ROOT_PASSWORD
                      value: password
                  ports:
                    - containerPort: 3306
                      name: mysql-board
            volumes:
                - name: mysql-board
---
apiVersion: v1
kind: Service
metadata:
    name: mysql-msa-board
spec:
    type: NodePort
    ports:
      - port: 3306
        protocol: TCP
        targetPort: 3306
        nodePort: ${MYSQL_MSA_BOARD}    #외부 포트. ${MYSQL_MSA_BOARD}: 305xx
    selector:
        app: mysql-board
```


## 참고영상
<https://www.youtube.com/watch?v=A1euGLsY4qE&list=PL-AoIAa-OgNncmEDFYsy0NAGvFu15zJb1&index=27>