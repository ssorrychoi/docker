---
layout: post
title: Java Secure coding_4
subtitle:
tags: [DB보안]
comments: true
---

20190725

## 👨🏼‍💻쿠키와 세션

### 쿠키

- 1994년 네스케이프에서 개발
- 웹 서버가 웹 브라우저로 전송하는 작은 텍스트 정보 파일
- 한번 저장된 쿠키는 *유효기한이 경과하지 않았다면 해당 도메인에 접속하면 브라우저가 자동으로 탑재하여 전송*
- *웹 서버는 Set-Cookie 헤더를 통해 생성한 쿠키를 클라이언트에게 전송*
- *웹 브라우저는 Cookie 요청 헤더에 쿠키를 더해서 웹 서버에서 요청*

- 설정항목
  - Expire : 유효기간(년,월,일)
  - MaxAge : 지속기간(지금부터 몇일/몇달 이내)
  - Secure : 보안속성 
  - HTTPOnly : 이 속성이 설정되면 쿠키는 클라이언트의 자바스크립트로 직접 접근할 수 없음(모든 브라우저가 지원하지 않음)
- 종류
  - 저장 기간 : 영구적(Persistent Disk), 반영구적(Non-Persistent, Memory) 
  - 보안 여부 : 보안(Secure, HTTPS), 비보안(Insecure, HTTP)
- 제한
  - 데이터 형식 : 문자열
  - 크기 : 4KB 이하
  - 변수 개수 : 쿠키 파일 하나 당 20개
- 쿠키 파일
  - 형태 : 사용자이름@접속한사이트
  - 내용 : 개발자가 설정한 내용
  - Default directory : C:₩Documents and Settings₩사용자이름₩Cookies

### 세션

- 웹 서버가 해당 웹 서버에 접근한(요청한) 클라이언트(사용자)를 식별하는 방법

- Server Side 기술로 HTTP의 Stateless한 특성을 보완하기 위하여, 유일(Unique)한 세션 아이디(Session ID)를 생성하여 현재 접속한 클라이언트에게 할당해 주고 기억하는 방식

- 웹 서버 또는 웹 어플리케이션 서버에서 세션 핸들링 서비스를 지원

- 세션 정보 가져오기

  ```java
  HttpSession session = request.getSession(true);
  HttpSession session = request.getSession();
  HttpSession session = request.getSession(false);
  ...
  if(session.isNew()){
    //새로 만들어진 세션이다.
  }
  ```

- 세션에 데이터 저장하기

  ```java
  session.setAttribute("username","Hong");
  ```

- 세션에서 데이터 읽어오기

  ```java
  String username = (String) session.getAttribute("username");
  ```

- 세션의 유효기간 설정

  *프로그램에서 세션 타임아웃 설정하기*

  ```java
  session.setMaxInactiveInterval(60*60);	//1시간으로 설정
  ```

  *웹 서버의 설정 파일(web.xml)에서 세션 타임아웃 설정하기*

  ```xml
  <session-config>
  	<session-timeout>30</session-timeout>		<!--디폴트 30분으로 설정-->
  </session-config>
  ```

- 세션 삭제

  ```java
  session.removeAttribute("username");   //username으로 저장된 데이터 삭제
  session.invalidate();									 //모든 세션 정보 삭제
  ```

## 👨🏼‍💻인코딩 스키마

### 문자 인코딩

- 문자들의 집합을 컴퓨터에서 저장하거나 통신에 사용할 목적으로 부호화하는 방법을 가리키며, 그냥 인코딩이라고도 불린다.

- 웹 애플리케이션에서 사용되는 데이터들도 네트워크를 통해 여러 종류의 시스템으로 전달되는 환경이므로 여러가지 인코딩 스키마를 적용하여 데이터를 안전하게 사용하도록 조치한다.

- 종류

  - **ASCII ( American Standard Code for Information Interchange )**

  - **URL Encoding** 

    - URL은 US ASCII 문자 집합에서 출력 가능한 문자들만 포함

    - URL Encoding은 URL 메타 문자(URL에서 제어 기능을 가진 문자)를 인코딩할 때 사용

    - 형식 : "%" + 기존 문자열의 HEX값

      | URL메타문자 | 설명                                  | URL 인코딩 |
      | :---------: | ------------------------------------- | :--------: |
      |      ~      | 사용자의 홈 디렉토리 접근시 사용      |            |
      |      @      | 메일 계정 표시                        |            |
      |      #      | 동일한 페이지 내에서 위치 이동시 사용 |    %23     |
      |      &      | 파라미터 구분자                       |    %26     |
      |      %      | HEX 값 표현에 사용                    |    %25     |
      |      +      | 공백문자                              |    %2b     |
      |      =      | 파라미터 대입 연산자                  |    %3d     |
      |      ?      | URL과 파라미터 구분자                 |    %3f     |
      |     ://     | 프로토콜 구분자                       |            |

  - **HTML Encoding** - *서버에서 만든 데이터를 그대로 내려받기 위해 사용. XSS공격 방어 기법*

    - 문제가 있을만한 문자들을 HTML 문서 컨텐츠의 일부분으로 안전하게 사용하기 위해 사용
    - HTML 문서 안에서 특별한 의미를 지니고, 문서의 구성 보다는 내용을 정의하는데 사용
    - HTML 엔티티 정의에 사용
    - 10진법, 16진법 등으로 표현 가능

  - **BASE64 Encoding**

    - 8bit 이진 데이터를 문자코드에 영향을 받지 않는 ASCII 문자열로 바꾸는 인코딩 방식
    - SMTP를 통해 이메일 첨부 파일을 전송하거나, HTTP 기본 인증에 사용
    - 6bit -> 64개의 문자를 표현
      - 0~25 -> A~Z, 26~51 -> a~z, 52~61 -> 0~9, 62 -> +, 63-> /
      - = -> 마지막 패딩 문자열

  ### *실습*1. Cookie & Session

  hello.jsp - 기본

  ```jsp
  <%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
  <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
  <html>
    <head>
      <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
      <title>Insert title here</title>
    </head>
    <body>
      <%	//스크립틀릿 : JSP에서 동적 처리하는 부분을 기술
  
      //요청 파라미터 목록으로부터 이름이 NAME인 파라미터의값을 문자열로 가져옴
      String name = request.getParameter("name");
      if(name == null || name.equals("")){		//파라미터가 없거나 값이 없으면
        %>
      <form method="POST" action="hello.jsp">
        <input type="text" name="name" value="">
        <input type="submit" value="안녕">
      </form>
      <%
      }else{
        out.print(name + "안녕 :)");
      }
      %>
    </body>
  </html>
  ```

  hello.jsp - Cookie

  ```jsp
  <body>
    <% 	// 스크립틀릿 : JSP에서 동적 처리하는 부분을 기술
  
    // 요청 파라미터 목록으로부터 이름이 name인 파라미터의 값을 문자열로 반환 
    String pname = request.getParameter("name");
    if (pname == null || pname.equals("")) {
      %>
    <form action="hello.jsp" method="post">
      <input type="text" name="name" value="">
      <input type="submit" value="안녕">
    </form>		
    <%		
    } else {
      // 파라미터로 전달된 값을 쿠키에 저장 및 화면 출력
      Cookie c = new Cookie("cname", pname);
      c.setDomain("localhost");
      c.setPath("/openeg");
      c.setMaxAge(60*60*24);
      response.addCookie(c);
    }
    %>	
  
    <a href="hello2.jsp">또 안녕</a>
  
  </body>
  ```

  hello2.jsp - Cookie

  ```jsp
  <body>
    <% 	
    String cname = "";
    Cookie[] cs = request.getCookies();
    for (int i = 0; i < cs.length; i++) {
      if ("cname".equals(cs[i].getName())) {
        cname = cs[i].getValue();
      }
    }
    %>
    <br>쿠키로부터 >>> <%=cname%> 또 안녕~
  </body>
  ```

  hello.jsp - Session

  ```jsp
  <body>
    <% 	// 스크립틀릿 : JSP에서 동적 처리하는 부분을 기술
  
    // 요청 파라미터 목록으로부터 이름이 name인 파라미터의 값을 문자열로 반환 
    String pname = request.getParameter("name");
    if (pname == null || pname.equals("")) {
      %>
    <form action="hello.jsp" method="post">
      <input type="text" name="name" value="">
      <input type="submit" value="안녕">
    </form>		
    <%		
    } else {
      // 파라미터로 전달된 값을 쿠키에 저장 및 화면 출력
      Cookie c = new Cookie("cname", pname);
      c.setDomain("localhost");
      c.setPath("/openeg");
      c.setMaxAge(60*60*24);
      response.addCookie(c);
  
      // 세션에 저장
      session.setAttribute("sname", pname);
      out.print(pname + " 안녕!!!");		
    }
    %>	
    <a href="hello2.jsp">또 안녕</a>
  </body>
  ```

  hello2.jsp - Session

  ```jsp
  <body>
    <% 	
    String cname = "";
    String sname = "";
    Cookie[] cs = request.getCookies();
    for (int i = 0; i < cs.length; i++) {
      if ("cname".equals(cs[i].getName())) {
        cname = cs[i].getValue();
      }
    }
    
    sname = (String)session.getAttribute("sname");
    %>
    <br>쿠키로부터 >>> <%=cname%> 또 안녕~
    <br>세션으로부터 >>> <%=sname%> 또 안녕~
  </body>
  ```

  ### *실습*2. URL Encoding

  ```jsp
  <%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
  <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
  <html>
    <head>
      <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    </head>
    <body>
      <%
      String pcompany = request.getParameter("company");
      if (pcompany == null) {
        pcompany = "";
      }
      out.print("회사이름은 " + pcompany + "입니다.");
      %>
      <form>
        <input type="text" name="company" value=""/>
        <input type="submit"> 
      </form>
  
      <a href="?company=<%=pcompany%>">링크로 접근</a>
      <a href="?company=<%=java.net.URLEncoder.encode(pcompany, "UTF-8")%>">링크로 접근</a>
    </body>
  </html>
  ```

  ### *실습*3. HTML Encoding

  ```jsp
  <%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
  <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
  <html>
    <head>
      <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    </head>
    <body>
      <%
      String phtml = request.getParameter("html");
      if (phtml == null) {
        phtml = "";
      }
      out.print("입력한 내용은 " + phtml + "입니다.");
      %>
      <form>
        <input type="text" name="html" value=""/>
        <input type="submit"> 
      </form>
    </body>
  </html>
  ```

  

## 정규식

- 문자열의 검색, 치환, 추출을 위해 사용되는 패턴
- 내가 원하는 데이터를 표현하는 방식
- **JavaScript에서 정규식 사용하기**
  - JavaScript에서 정규식을 사용하기 위해서는 정규식을 처리할 RegExp 객체를 생성한다.
- **Java에서 정규식 사용하기**
  - Java 프로그램에서 정규식을 사용하기 위해서는 Pattern, Matcher 클래스를 사용한다.

| 정규식   | 의미                                            | 예                                                           |
| -------- | ----------------------------------------------- | ------------------------------------------------------------ |
| .        | (\n 문자를 제외한)<br />한 문자가 일치하는 검사 | ab. : ab 다음에 아무 문자든 하나가 있다.                     |
| [ ]      | [abc]test<br />atest,btest,ctest                | [ ] 안의 형식과 일치하는 한 문자<br />첫 번째 글자가 a이거나 b이거나 c다. |
| [0-9]    | ab[0-9]<br />ab1,ab2,ab3 ...                    | [부터 -까지] 의 범위 내 숫자 하나<br />0-9까지 한 자리 숫자  |
| [a-z]    | ab[a-z]<br />aba,abb,abc, ...                   | [부터-까지]의 범위 내 소문자 한 문자<br />a-z까지 한 자리 문자 |
| [a-zA-Z] | ab[a-z]<br />aba, abA,abZ, ...                  | [부터-까지]의 범위 내 대소문자 한 문자<br />모든 알파벳 한 문자 |
| [^]      | [^ab]cd<br />ecd, fcd, gcd                      | ^이후의 괄호 안 문자를 제외한 한 문자<br />첫 번째 글자가 a나 b가 아닌 문자 |

***정규식 표현 테스트***

```
핸드폰번호 정규식
https://regexpal.com/

1. [0-9][0-9][0-9]-[0-9][0-9][0-9][0-9]-[0-9][0-9][0-9][0-9]
2. \d\d\d-\d\d\d\d-\d\d\d\d
3. \d{3}-\d{4}-\d{4}
4. 01[016789]-\d{4}-\d{4}
5. 01(?:0|1|[6-9])-\d{4}-\d{4}
```

# 보안취약점 제거를 위한 코딩 기법

## 1. 입력값 검증 부재와 삽입

### 인젝션 ->막기위해선 입력값 검증을 해야한다.

- 입력 값에 대한 확인 절차가 생략되는 경우 다양한 인젝션(삽입) 취약점이 발생할 수 있다.
  - DB데이터를 조작하는 SQL문에 검증되지 않은 외부 입력값을 사용하는 경우
  - 내부에서 실행되는 명령어나 명령어의 인자(argument)로 검증되지 않은 외부 입력값 사용
  - LDAP 조회를 위한 필터 조립에 검증되지 않은 외부 입력값 사용
  - XPath 쿼리 작성에 검증되지 않은 외부 입력값 사용
  - Query 쿼리 작성에 검증되지 않은 외부 입력값 사용
  - SOAP 서비스 요청 메세지 작성에 검증되지 않은 외부 입력값 사용



- **SQL Injection**

  - 웹 애플리케이션에서 입력 받아 데이터베이스로 전달하는 정상적인 쿼리(CRUD)를 변조, 삽입하여 데이터베이스에 불법적인 데이터 열람, 시스템 명령 수행 등 비정상적인 데이터베이스에 접근하는 기법이다. 모든 종류의 DBMS에 적용 가능한 공격 기법이며, 지속적으로 발전되고 있는 공격 기법이다.

    ```java
    String ptext = request.getParameter("text");
    String query = "select * from data where keyword ='" + ptext + "'"; //검증없이 사용.
    
    Statement stmt = connection.createStatement();
    stmt.executeQuery(query);
    ```

    정상적인 요청 => search.jsp?text=abcd

    => `select * from data where keywod='abcd'`


    비정상적인 요청 => search.jsp?text='abcd' or 'a'='a

    => `select * from data where keyword='abcd' or 'a'='a'` 

  

  - 비정상적인 요청 유형 -> SQL Injection 공격 유형

  

  - 비정상적인 요청

    1) **항상 참이 되는 입력** => 모든 내용이 반환됨 = 권한 밖의 데이터에 대해 조회접근 및 조회가 가능해진다.

    ​	=> search.jsp?text='abcd' or 'a'='a

    ​	=> `select * from data where keyword='abcd' or 'a'='a'` 

    2)  **오류를 유발하는 입력** => 추가 공격을 위한 정보 수집 가능

    ​	=> search.jsp?text=abcd'

    ​	=> select * from data where keyword='abcd' '

    ​	=> 홑따움표의 개수가 일치하지 않아서 오류가 발생 -> 오류

    ​	=> 메세지에 대한 처리가 불완전하여 시스템 내부 정보가 사용자 화면에 출력될 수 있음.

    

  - 취약점 발생원인 : 웹 애플리케이션의 SQL 쿼리문