## 위협 모델링 (Threat Modeling)

### 위협 모델이란?

- 위협 모델은 설계 단계에서 보안 문제에 대한 체계적인 접근에 도움을 주기 위한 것이다.
- 위협 모델은 제품에 존재하는 가장 취약한 보안 위험이 무엇이며, 공격자가 이를 어떻게 이용할 수 있는가에 대한 분석 방법이다.
- 위협분석의 목적은 대응해야 할 위협을 파악하고, 어떻게 대응할 수 있는지를 결정하기 위한 체계적인 방법을 제공하기 위한 것이다.

### 위협 모델링 7단계

1) 위협 모델링을 수행할 팀을 소집한다.
2) 애플리케이션을 분해한다.
3) 시스템에 대한 위협 결정
4) 위협 우선순위 결정
5) 대응 방법 결정
6) 적용할 기법/기술을 선택
7) 

### Seven Touchpoints

-  실무적으로 검증된 방법론 중 하나
- SDLC( Software Development Life Cycle )내의 개발 단계와 이와 관련된 7개의 보안강화활동을 개발자에게 집중적으로 관리하도록 요구

### OWASP CLASP

- Secure Software 사에서 개발
- SDLC 초기단계의 보안강화에 목적을 둔 정형 프로세스
- 활동중심, 역할기반의 프로세스로 구성된 집합체
- View - Activity - Process - Component로 구성

# Part2. 안전한 소프트웨어를 만드는 시큐어코딩 기법

## 

### 실습환경 구축

- Open Virtual Machine 클릭 > 이미지 3개 등록 (Victim, Attacker, WindowXP)
  - Kali #1 -> 취약한 웹 서버, PHP 기반의 웹 서비스 2개가 동작
  - Kali #2 -> 공격자용 PC, Kali Linux에서 제공하는 기능(도구)를 이용해서 공격에 활용
  - WinXP -> 취약한 웹 서버, Windows기반에 기능(도구)를 이용해서 공격에 활용
- 로그인 계정
  - Kali linux : root / toor
  - Windows XP : administrator / 0sook

### Kali#1 Victim ip :  *192.168.111.130*

### Kali#2 Attacker ip : *192.168.111.131*

### WinXP ip : *192.168.111.129*

### HOST PC ip : *70.12.50.52* or VMware VMnet 8 ip : *192.168.111.1*

@WinXP에서 paros를 실행한다.
==> paros를 실행
==>  제어판 > 인터넷옵션 > 프록시 사용 지정(8081포트로 대기) 
==> IE 브라우저에 Cooxie 툴바의 proxy 선택창에서 127.0.0.1:8081를 선택

1.  @Kali#1에서 mysql & apache2 실행

   `service mysql start`

   `service apache2 start`

   

2. @Kali#2, @WinXP에서 Kali의 웹 사이트로 접속

   ⇒ http://KALI#1_IP/

   ⇒ http://KALI#1_IP/bWAPP (bee / bug)

   

## HTTP와 웹 구조

p.131

###HTTP 

- Stateless , Connectionless 하다. => 연결을 유지하지 않는다.
- 요청 / 응답 구조를 갖는다.

**Request**

| 메세지 구조 | 내용                                                     |
| :---------: | -------------------------------------------------------- |
|    시작     | 요청방식(Method), 요청 페이지(URI), 프로토콜/버전        |
|    헤더     | General Headers<br />Request Headers<br />Entity Headers |
| Blank Line  |                                                          |
|    Body     | 서버로 전달할 Parameter                                  |

**Response**

| 메세지 구조 | 내용                                                      |
| :---------: | --------------------------------------------------------- |
|    시작     | 프로토콜/버전 , 응답 상태 코드 , 응답 상태 메세지         |
|    헤더     | General Headers<br />Response Headers<br />Entity Headers |
| Blank Line  |                                                           |
|    Body     | 응답 메세지(HTML)                                         |



### NC를 이용한 HTTP 메소드 테스트

문제 1. Kali#2에서 HTTP 메소드를 이용해서 openeg 사이트의 hello.html 파일을 삭제해 보세요.

```
root@kali:~# nc 192.168.111.1 8080
DELETE /hello.html HTTP/1.0

HTTP/1.1 204 No Content
Server: Apache-Coyote/1.1
Date: Wed, 24 Jul 2019 08:31:55 GMT
Connection: close
```

문제 2. Kali#2에서 HTTP PUT 메소드를 이용해서 openeg 사이트의 hello.html 파일을 2번 생성해 보세요.

```
root@kali:~# nc 192.168.111.1 8080
PUT /hello2.html HTTP/1.0
Content-Type: text/html
Content-Length: 42

<html><body>Hello, World!!</body></html>

HTTP/1.1 201 Created
Server: Apache-Coyote/1.1
Content-Length: 0
Date: Wed, 24 Jul 2019 08:40:16 GMT
Connection: close

root@kali:~# nc 192.168.111.1 8080
PUT /hello2.html HTTP/1.0
Content-Type: text/html
Content-Length: 42         

<html><body>Hello Jaeseong!</body></html>
HTTP/1.1 204 No Content
Server: Apache-Coyote/1.1
Date: Wed, 24 Jul 2019 08:42:56 GMT
Connection: close

```

