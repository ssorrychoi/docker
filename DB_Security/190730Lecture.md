## XSS (크로스 사이트 스크립팅)

- 공격자가 전달한 스크립트 코드가 사용자 브라우저를 통해서 실행되는 것
  => 사용자 브라우저 또는 사용자 PC의 저장된 정보를 탈취
  => 가짜 페이지를 만들어 사용자로 하여금 추가 입력을 유도하고, 해당 정보를 탈취
  => 사용자 PC를 좀비화하여 원격에서 해당 PC를 조장 ->(BeEF Framework를 사용)

  

  1) 공격자가 전달한 스크립트 코드가 취약점이 있는 해당서버를 경유해서 사용자 브라우저에게 사용되는 방법 (Reflective XSS)

  ​	=> 입력값이 입력값 검증이나 출력값 검증 없이 다음 화면 출력에 그대로 사용되는 경우에 발생

  ```java
  안녕! <%= request.getParameter("input")%>
  ====================================
  <%
  	out.print("안녕! "+request.getParameter("input"));
  %>
  
  정상적인 입출력  : ..../do.jsp?input=홍길동       =>안녕! 홍길동
  비정상적인 입출력 : ..../do.jsp?input=<script>alert(document.cookie)</script>    => 안녕! alert();->해당 브라우저의 쿠키값이 출력된다.
  ```

  2) 저장(Stored XSS)

  - 공격자가 작성한 스크립트 코드가 취한한 서버에 저장되어 기반적으로 사용 브라우저로 내려가서 실행되는 것 => 크스립트 코드를 만들어 게시판 글쓰기 페이지에 소스를 넣는 것.

  3) DOM Based XSS

  - 개발자가 작성한 스크립트 코드의 취약점을 이용한 공격 기법
  - document.write( )