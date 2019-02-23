# Instance 부하 측정

## nGrinder 설치

### Java설치
```
$yum install java-1.8.0-openjdk-devel.x86_64
```
  

### Java 환경변수 설정
```
$vi /etc/profile
```
  
```
export JAVA_HOME=/usr/share/jdk8
export PATH=$PATH:$JAVA_HOME/bin
```
  
```
$source /etc/profile //Apply Enviromental Variable
```
### Docker 설치 및 실행
```
$curl -fsSL  [https://get.docker.com/](https://get.docker.com/)  | sudo sh

$systemctl start docker.service
```
### Tomcat 설치

```
$wget http:``//archive``.apache.org``/dist/tomcat/tomcat-7/v7``.0.50``/bin/apache-tomcat-7``.0.50.``tar``.gz` `--2016-04-05 14:44:49-- http:``//archive``.apache.org``/dist/tomcat/tomcat-7/v7``.0.50``/bin/apache-tomcat-7``.0.50.``tar``.gz`
```
  

### nGrinder Controller 설치 및 실행
```
$wget http://downloads.sourceforge.net/project/ngrinder/ngrinder-3.3/ngrinder-controller-3.4.1.war
```
  

### http://IP_ADDRESS:8080으로 Controller Login

* 초기 로그인 ID / PW : admin / admin

  

### nGrinder agent 다운로드 후 설치
```
$docker run --name ngrinder_agent -d -e CONTROLLER=ADDR=(IP_ADDRESS):#PORT ngrinder/agent:3.3
```
## Script 생성 및 Test 수행
### Performance Test 창에서 Test 수행

### Apache

  

* Agent : 1

* vUsers : 500

  

* 5회분 측정 결과 : 1,062.5 / 1,037.3 / 1,046.8 / 1,036.7 / 1,027.4

* Avg of TPS : 1,042.14

  


### nginx

  

* Agent : 1

* vUsers : 400 (apache에 비해 할당할 수 있는 vUsers 최대치가 적음)

  

* nginx 서버 가동 후 처음 3회차 측정 테스트 시 vUsers를 500까지 할당할 수 있었으나, 이후의 테스트에선 불가하여 vUsers의 숫자를 400으로 내림

  

### 할당 가능한 vUsers가 적은 이유

* 테스트 결과의 특별한 error log는 남지 않았으나, 측정 중 Agent State를 감시한 결과

  

* vUsers를 500으로 할당하였을 때 Agent의 MEM 사용량이 94%에서 더 높아지며 Controller가 Killed 상태로 변경됨

* vUsers를 400으로 할당하였을 때 Agent의 MEM 사용량이 94%를 정점으로 더이상 높아지지 않음

  

* 4회분 측정 결과 : 1,013.6 / 1,035.2 / 1,020 / 995.7

* Avg of TPS : 1,016.125


### LBaaS에 Apache Server 연결 후 측정

  

* Agent : 1

* vUsers : 500

  

* 5회분 측정 결과 : 974.5 / 995.9 / 1,012.4 / 1,036.6 / 1,028.2

* Avg of TPS : 1,009.52

  

### LBaaS에 NGINX Server 연결 후 측정

  

* Agent : 1

* vUsers : 400 (apache에 비해 할당할 수 있는 vUsers 최대치가 적음)

  

* 5회분 측정 결과 : 1,038.7 / 1054 / 1,036.1 / 1,076.7 / 1,049.7

* Avg of TPS : 1,051.04

  

### LBaaS에 인스턴스를 연결했을 때 TPS 변화가 없는 이유
  

* Load Balancer에 부하를 줄 때 Load Balancer 자체에서 부하를 조정하고 증가할 수 있는 TPS의 제한이 걸린 것이라 예상됨


##  CDN 구축

  

* 서비스 지역 : KOREA / Cache 만료 설정 : 사용자 설정 / Cache 만료 시간 : 300

  

### CDN 구축 후 측정 - Instance 1개

### 테스트 설정(Apache, nginx 공통)
* Agents - 3
* vUsers - 500  


  

### HTTP Apache

  
* 5회분 측정 결과 : 28,888.2 / 29,046.8 / 27,450.8 / 28,589.4 / 29,396.5

* Avg of TPS : 28,666.34

  ![](https://lh3.googleusercontent.com/oJ1i1F-2eljnMYtMPLuFsETudESMxB2Ey7en4LiGslpJqAvCProG4R4ZhYTUHOBnrcXMVF-v6AXO)
  

### NGINX

  
* 5회분 측정 결과 : 11,708.6 / 11,705.4 / 11,502.9 / 11,632.6 / 11,494.2

* Avg of TPS : 11,608.74

* 특이사항 : 0.03% ~ 0.05%의 Err Rate 발생


![](https://lh3.googleusercontent.com/Kip-09haRnd-GtzUAqsbX5f7JDhoF-BX2VDjppmW66UzGnpYEIDDZbk1KIBhyclJcXlWt5nwEtwU)
  

**4. CDN 사용 시 효과**

  

### Apache 측정 결과

  

* CDN 도입 전 : 1,042.14 / CDN 도입 후 : 28,666.34

  

* 약 27.5배 성능 향상

  

### NGINX 측정 결과

  

* CDN 도입 전 : 1,016.125 / CDN 도입 후 : 11,608.74

  

* 약 11.4배 성능 향상

  

### CDN 도입 시 이점

* 컨텐츠가 캐시 서버에 위치 -> 증가하는 트래픽에 따른 서비스 접속 불가 문제 미연에 방지

  

* 클라이언트-서버 환경의 컨텐츠 전송을 위한 회선 비용 감소

  

* 빠른 Response Time으로 컨텐츠 제공, 병목현상 저하

  

### LBaaS 알고리즘 변경 후 TPS 변화량 측정 (NGINX 기반)


* 변경 전 : Round Robin (1,051.04)

  

* Least Connections - Avg of TPS 변화 미비 (1,053.46)

  

*  Source IP - 5회분 측정 결과 Avg of TPS 변화 미비 (1,049.56)

  

### LBaaS 알고리즘 변경 후 TPS 변화량 측정 (Apache 기반)

* 변경 전 : Round Robin (1,009.52)

  

* Least Connections - Avg of TPS 약 100 증가(1,101.8)

  

* 6-2. Source IP - Avg of TPS 소폭 증가 (1,078.428)

  

### LBaaS알고리즘 변경 후에도 TPS의 변화가 미비한 이유

  

* Least Connections : TCP 연결 수가 가장 작은 인스턴스를 선택하는 방식이지만, Load Balancer에 달린 인스턴스가 2개이기 때문에 최소 연결의 차이가 미비함

  

* Source IP : 원본 IP를 해싱하여 처리할 인스턴스를 선택하는 방식. 기존에 Load Balancer에 추가되어 있던 인스턴스의 이미지를 생성하여 추가 할당한 것이기 때문에, 두 인스턴스의 사설 IP주소는 동일한 대역에 있음

* 따라서, 해싱 값의 차이가 미비함

  

## Apache, nginx 튜닝을 통한 TPS 향상

  

### Apache

* 테스팅 한 버전(Apache 2.4.6)


* MPM 방식은 아파치 컴파일 시 지정할 수 있으며, 본 버전은 prefork(기본)으로 지정되어 있음

* prefork 방식은 쓰레드가 한 개인 자식 프로세스를 여러개 사용하는 방식으로, 많은 메모리를 사용함.

* 성능 최적화를 위해서는 /usr/share/doc/httpd-2.4.6/httpd-mpm.conf 상의 하기 수치를 수정해야 함.

```
StartServers
MinSpareServers
MaxSpareServers
MaxRequestWorkers
MaxConnectionsPerChild
```  

* 인스턴스 서버의 스팩에 따라, 최적화할 수 있는 수치가 다름

  

### NGINX

  

* nginx의 튜닝은 /etc/nginx/nginx.conf 파일 수정을 통해 가능함

* worker_process : auto가 default 이지만, 보통 인스턴스의 코어 수 만큼 할당함

* worker_connections : 하나의 프로세스가 처리할 수 있는 커넥션의 수

* worker_process * worker_connections = 최대 접속자 수

  

* NGINX 역시 인스턴스 서버의 스팩에 따라, 최적화할 수 있는 수치가 다름

  

  

## TPS 측정 시 서버쪽 리소스 사용량의 변화

  

### 측정 전

* usr(사용자 프로세스가 사용중인 CPU) 0 / sys(시스템 프로세스가 사용중인 CPU) 0 / idl(유휴 CPU) 100% / cach(Cache Mem) 634M / free(여유 공간) 924M

### 측정 후
* usr(사용자 프로세스가 사용중인 CPU) 1 ~3 범위에서 증감 반복 / sys(시스템 프로세스가 사용중인 CPU) 1 ~ 5 범위에서 증감 반복 / idl(유휴 CPU) 점진적 감소 / cach(Cache Mem) 점진적 증가

* free(여유 공간) 점진적 감소

  ![](https://lh3.googleusercontent.com/3AmEmm8WoD9v2vC0xcDIQshuXsct31MwOzOtxANkb3deq7gDrYBcspY59JKRKf6TckB9mkMWnPUd)