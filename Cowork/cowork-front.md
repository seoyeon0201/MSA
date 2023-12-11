# 1. EC2에 Front 배포
## 1. EC2 Instance 생성

- Instance Name: `cowork-front`
- OS Image: `Ubuntu`
- Storage: `30GiB`
- pem 키 경로 `C:\Cloud\cowork-front.pem`

## 2. 보안그룹 편집

- 기존 `ssh` 외의 `http`, `https` 추가
- 서비스 사용시 사용할 `8080 port`도 추가

![Alt text](image-29.png)

## 3. SSH를 이용한 EC2 Instance 접속

- window 기본 cmd가 아닌 `GitBash`로 접속
- 키페어(.pem)을 C:/Cloud로 복사
- 아래 명령어로 접속 > yes/no 선택 시 yes 선택
```
ssh -i C:/Cloud/cowork-front.pem ubuntu@[퍼블릭 IPv4 주소]   #[퍼블릭 IPv4 주소]는 xx.xx.xxx.xxx의 10자리 수
```

![Alt text](image-20.png)

## 4. 프론트엔드

### 1. npm 환경 구축

- 우분투에 `curl` 설치
`sudo apt-get install -y curl`

- node.js와 npm 설치
```
sudo apt update
sudo apt install nodejs
sudo apt-get install -y npm
sudo npm install -g npm@6   #sudo apt install npm은 최신 버전이 설치되는데 7 version 이상은 오류 존재하므로 6으로 설치

npm install vue    #본 프로젝트에서 vue 사용하기에 설치
npm install axios    #본 프로젝트에서 axios 사용하기에 설치

#위 명령어만 했는데 버전 확인 안 되는 경우
sudo apt-get update
sudo apt-get install -y build-essential
sudo apt-get install curl
curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash --
sudo apt-get install -y nodejs
```

- 설치 확인
```
node -v
npm -v
npm vue -v
```


### 2. git clone으로 github에서 배포할 파일 가져오기

### 3. 서버 배포

- 코드에서 axios 오류난 곳 코드 삭제

- .env 파일 확인
    - `ls -a`로 확인 가능
    - `gateway URI 정확히 입력`했는지 확인
        - gateway URI: `[KPAAS harbor IP]:[apigateway NodePort]`

![Alt text](image-27.png)

- git clone으로 받은 파일은 git에 올라온 이름으로 폴더 생성 
    - 해당 폴더로 들어가서 실행해야함
    - 본 폴더명은 `front`
```
cd front
npm i   #npm install의 약자. package.json 에 포함된 의존성 패키지 일괄 설치
npm run serve
```

- `[퍼블릭 IPv4 주소]:8080`

![Alt text](image-28.png)

# 2. Route53 도메인 인증

> (1)가비아에서 도메인을 구매하고, (2)AWS(Route 53)에서 도메인 소유를 인증하고,(3+4)ACM(AWS Certification Manager)를 통해 SSL(TSL) 인증서를 발급
> (5)EC2 인스턴스의 로드 밸런싱을 위한 타겟 그룹을 생성하고,(6)로드 밸런서(Load Balancer)로 리다이렉트 규칙을 설정하고(http 요청을 https로 리다이렉트),(7)로드 밸런서의 Health check를 통과해 로드 밸런싱을 안전하게 유지 

## 1. 가비아에서 도메인 구매

`testdomain2.store`

## 2. AWS Route53에서 도메인 소유 인증

1. `AWS Route53`의 호스팅 영역 > 호스팅 영역 생성
- 도메인 이름을 가비아에서 구매한 도메인으로 작성
- 나머지는 default로 유지

2. `가비아`의 도메인 네임서버 변경

- 가비아 홈페이지에서 My가비아 > 이용중인 서비스 > 도메인> (내가 구입한 도메인)관리 클릭

- 네임서버 옆의 "설정" 클릭
    - 1~4차 호스트명에 AWS Route53 레코드의 NS 레코드의 라우팅 대상 4개 삽입
        - 이때 마지막 "."은 제거하고 삽입
    - 소유자 인증 후 적용

## 3. ACM을 통해 SSL 인증서 발급

- `SSL`(TSL) 인증서 발급
    - 도메인에 대한 보안 인증서
1. AWS에 `Certificate Manager` 검색해 대시보드로 이동
- 인증서 요청 > 퍼블릭 인증서 요청  
    - 도메인 이름에 구매한 도메인 입력
    - DNS 검증 (기본)
    - RSA 2048 (기본) 설정 후 요청
2. ACM 메뉴(왼쪽)에서 "인증서 나열" 클릭 시 요청한 인증서 확인 가능
- DNS 검증 상태가 `검증 대기중`, `아니요`, `부적격`이 정상

## 4. CNAME 레코드 생성
1. 방금 발급받은 인증서 ID 클릭 > "도메인에서 Route53에서 레코드 생성" 클릭
- 도메인 체크
2. 일정 시간 지난 후 인증서 상태가 `발급됨`으로 변경
- 짧게 20~30분 소요되지만 길게 2시간까지도 소요

## 5. EC2 인스턴스의 로드 밸런싱을 위한 타겟 그룹을 생성

1. 


## 6. 로드 밸런서(Load Balancer)로 리다이렉트 규칙을 설정하고(http 요청을 https로 리다이렉트)

## 7. 로드 밸런서의 Health check를 통과해 로드 밸런싱을 안전하게 유지 


#### 참고
<https://woojin.tistory.com/93#google_vignette>
<https://woojin.tistory.com/94>

### 오류 Collection

## 1. EC2에 Front 배포

**1. Daemons using outdated libraries**

![Alt text](image-22.png)

**해결**
```
#needrestart.conf 구성 파일 확인
vim /etc/needrestart/needrestart.conf 

#설정 변경
echo "\$nrconf{restart} = 'l';" | sudo tee /etc/needrestart/needrestart.conf    

#패키지 검색 후 needrestart 키워트 필터링해 목록 확인
dpkg -l | grep needrestart  

#-b: needrestart를 실행할 때, 재시작이 필요한 프로세스를 실제로 재시작하지 않도록 하는 옵션, -v: 상세한 출력 보여주는 옵션
needrestart -b -v   
```

![Alt text](image-23.png)

**2. @vitejs/plugin-vue requires vue (>=3.2.13) or @vue/compiler-sfc to be present in the dependency tree.**

**해결**
- `프로젝트 내`에서 아래 코드 실행
```
npm install vue@3.2.13 --save-dev
npm i @vue/compiler-sfc #@vue/compiler-sfc : 브라우저에서 vue파일이 작동하도록 변환
#npm install @vue/complier-sfc --save-dev
```

**3. A complete log of this run can be found in**

![Alt text](image-24.png)
**해결**
- ec2에 접속했는지 확인 후 접속되지 않았으면 접속
- 캐시 삭제 후 재설치
```
npm cache clean --force 
npm install --cache
```

**4. npm ERR! path /home/ubuntu/front-test/front/node_modules/node-sass npm ERR! command failed**

![Alt text](image-26.png)

**해결**

- node-sass를 최신 버전으로 재설치
```
npm add node-sass
```

**5. 무한로딩**

**해결**

- npm version 7이상에 문제가 있는듯해서 npm version6으로 재설치
- npm 삭제 시 nodejs도 함께 삭제 후 재설치

```
sudo apt-get remove nodejs
sudo apt-get remove npm

#cd /etc/apt/sources.list.d로 이동하여 노드 목록 제거

sudo rm -rf /usr/local/bin/npm /usr/local/share/man/man1/node* /usr/local/lib/dtrace/node.d ~/.npm ~/.node-gyp /opt/local/bin/node /opt/local/include/node /opt/local/lib/node_modules

sudo rm -rf /usr/local/lib/node*

sudo rm -rf /usr/local/include/node*

sudo rm -rf /usr/local/bin/node*
```

- 확인
```
node -v
npm -v
```


### 개념

`SSH`는 네트워크 상 다른 컴퓨터의 쉘을 사용할 수 있게 해 주는 프로그램 혹은 그 프로토콜

`pem` 형식의 파일은 우리가 생성한 서버에 원격으로 접속할 때, 외부의 보안 위협으로부터 보호해주는 ‘SSH’라는 보안 방식이 적용된 서버에서 반드시 필요한 파일

## 2. 