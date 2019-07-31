## XSS 방어 

TestContorller.java

```java
public String testXss(HttpServletRequest request) {
  StringBuffer buffer = new StringBuffer();
  //입력값을 검증없이 출력으로 반환
  // /test/xss_test.do?data=<script>...</script> 형식으로 호출되면
  // <script>...</script> 가 브라우저로 전달되는 구조

  // 방어대책 : 입력값을 HTML인코딩하여 출력값으로 반환
  String data = request.getParameter("data");
  //방법1. 모든 태그에 대해 일괄되게 HTML인코딩을 적용
  if(data != null){
    data = data.replaceAll("<","&lt;");
    data = data.replaceAll(">","&gt;");
  }
  buffer.append(data);
  return buffer.toString();
}
```

입력 : 
1)`<script>alert("XSS")</script>` 
2) `<u>XSS</u>` 

출력 : 
1) \<script>alert("XSS")\</script>
2) \<u>XSS\</u>

따라서 선별적으로 태그를 막기 위해선 검증된 Library를 사용하는 것이 필요하다. 내가 혼자 다 막을려고 하면 집에 못감.
NAVER에서 만든 **LUCY-XSS** 라이브러리를 사용해보자.

TestController.java

```java
public String testXss(HttpServletRequest request) {
		StringBuffer buffer = new StringBuffer();
		String data = request.getParameter("data");
		
		//방법 2. lucy-xss와 같은 검증된 필터 라이브러리를 사용
		// 1) jar 파일을 프로젝트 라이브러리에 등록
		//		C:\FullstackLAB\download\lucy-xss-filter\lucy-xss-1.1.2.jar
		//		openeg > Webcontent > WEB-INF > lib
		// 2) xml 파일(rule set)을 소스 디렉터리에 복사
		//		C:\FullstackLAB\download\lucy-xss-filter\lucy-xss-superset
		//		openeg > Java Resources > src
		
		// 인스턴스 생성 후 필터링
		XssFilter filter = XssFilter.getInstance("lucy-xss-superset.xml");
		data = filter.doFilter(data);
		
		buffer.append(data);
		return buffer.toString();
	}
```

입력 :
1) `<script>alert("XSS")</script>` 로 입력했을때 문자열 그대로 출력
2) `<u>underline</u>` 로 입력했을때 <u>underline</u> 으로 출력됨.

출력 : 
1) \<script>alert("XSS")\<script>
2) <u>underline</u>



BoardController.java

```java
@RequestMapping("/view.do")
public ModelAndView boardView(HttpServletRequest request) {
  int idx = Integer.parseInt(request.getParameter("idx"));
  //DB에서 읽어온 게시판 정보가 board라는 객체에 저장
  BoardModel board = service.getOneArticle(idx);

  // board 객체에서 게시판 내용을 추출하여 (lucy를 이용한) HTML 인코딩 후 다시 저장
  String content = board.getContent();
  XssFilter filter = XssFilter.getInstance("lucy-xss-superset.xml");
  content = filter.doFilter(content);
  board.setContent(content);


  service.updateHitcount(board.getHitcount() + 1, idx);

  List<BoardCommentModel> commentList = service.getCommentList(idx);

  ModelAndView mav = new ModelAndView();
  mav.addObject("board", board);
  mav.addObject("commentList", commentList);
  mav.setViewName("/board/view");
  return mav;
}

```

view.jsp

```jsp
<td colspan="4" align="left">
  <%-- 
  ${board.content}
  --%>
  <c:out value="${board.content}" />
</td>
```

- Tag Library를 이용해서 출력할것.



## 크로스 사이트 요청 위조(CSRF)

- 서버로 전달된 요청을 요청 절차와 요청 주체를 검증하지 않고 처리했을 때 발생.
- 공격자가 심어 놓은 코드를 통한 자동화된 요청이 희생자의 권한으로 실행되게 된다.

### 패스워드 변경 신청 폼                   ———>                           패스워드 변경 처리

changePwForm.jsp																			changePwProc.jsp

​																					1)  ChangePwProc.jsp인증1) 인증(로그인) 여부를 판단

​																					2) 변경에 필요한 정보(newPw) 가 포함되었는지 확인

​																					3) 세션 -> 사용자 정보(userId)

​																					4) update userd,PW 

### 방어대책

1) 요청 절차를 확인한다.
	a) Referer 요청 헤더를 검증
	b) Token을 검증
	c) CAPTCHA, reCAPTCHA를 검증 = 자동화된 요청을 방지 = 사용자와의 상호작용을 통한 요청 처리

2) 요청 주체를 확인한다.
	a) 주요 기능에 대해 재인증, 재인가 후 처리한다.
		➖  정보가 생성, 수정, 삭제되는 경우 = 트랜젝션이 발생
		➖  중요 정보를 다루는 기능
		➖  과금이 발생하는 기능



P.279

### CSRF 취약점을 이용한 게시판 자동 글쓰기

- 공격자는 게시판 글쓰기 요청 페이지에서 어떤 값들이 서버로 전달되는지 분석

```java
<form action="write.do" method="post" enctype="multipart/form-data">
<input type="text" name="subject" value="저렴한 대출상품 안내"/> 
<input type="hidden" name="writer" value="관리자" /> 
<input type="hidden" name="writerId" value="admin" />
<textarea name="content">무담보 대출 상품 안내 연락주세요...</textarea>
<input type="submit" id="btnSubmit"/>
</form>
<script>document.getElementById("btnSubmit").click();</script>
```

=> 코드를 게시판 "내용"에 등록.

### 방어코드

BoardController.java > boardWrite()

```java
//  글쓰기 화면(페이지)을 요청받으면 글쓰기 화면을 반환
@RequestMapping("/write.do")
public String boardWrite(@ModelAttribute("BoardModel") BoardModel boardModel, HttpSession session) {
  // 임의의 토큰을 생성한 후 세션에 저장
  String token = UUID.randomUUID().toString();
  session.setAttribute("stoken", token);

  return "/board/write";
}
```

write.jsp

```jsp
<form action="write.do" method="post" onsubmit="return writeFormCheck()" enctype="multipart/form-data">
  <%-- 
  서버에서 생성 후 세션에 저장해 둔 토큰값을 히든 필드의 값으로 설정
	--%>
  <input type="hidden" name="ptoken" value="${stoken}" />
```

BoardController.java > boardWriteProc()

```java
@RequestMapping(value = "/write.do", method = RequestMethod.POST)
public String boardWriteProc(@ModelAttribute("BoardModel") BoardModel boardModel, MultipartHttpServletRequest request, HttpSession session) {

  // 파라미터로 전달된 토큰값과 세션에 저장된 토큰값을 비교하여
  // 일치하는 경우에는 게시물을 저장하고, 
  // 일치하지 않는 경우에는 list.do로 리다이렉트 한다.
  String sToken = (String)session.getAttribute("stoken");
  String pToken = request.getParameter("ptoken");
  if (pToken == null || !pToken.equals(sToken)) {
    return "redirect:list.do";
  }

  String uploadPath = session.getServletContext().getRealPath("/") + "files/";
  File dir = new File(uploadPath);
  if (!dir.exists()) {
    dir.mkdir();
  }

  MultipartFile file = request.getFile("file");
  if (file != null && !"".equals(file.getOriginalFilename())) {
    String fileName = file.getOriginalFilename();
    File uploadFile = new File(uploadPath + fileName);
    if (uploadFile.exists()) {
      fileName = new Date().getTime() + fileName;
      uploadFile = new File(uploadPath + fileName);
    }

    try {
      file.transferTo(uploadFile);
    } catch (Exception e) {
      System.out.println("upload error");
    }

    boardModel.setFileName(fileName);
  }

  // 진단 
  // 클라이언트에서 전달된 게시판 내용에 
  // 실행 가능한 스크립트 코드 포함 여부를 확인하지 않고
  // DB에 저장하고 있음 
  // --> DB에 저장된 스크립트 코드가 지속적으로 사용자 브라우저로 
  //     전달되어 실행되게 됨 
  // ==> Stored XSS 취약점을 가지고 있다.

  // 대응
  // 클라이언트에서 전달된 게시판 내용에 포함된 
  // 실행 가능한 스크립트를 HTML 인코딩한 후 DB에 저장하도록 수정.

  // 게시판 내용을 추출
  //		String c = boardModel.getContent();
  //		XssFilter filter = XssFilter.getInstance("lucy-xss-superset.xml");
  //		c = filter.doFilter(c);
  //		boardModel.setContent(c);		

  String content = boardModel.getContent().replaceAll("\r\n", "<br />");
  boardModel.setContent(content);
  service.writeArticle(boardModel);

  return "redirect:list.do";
}
```



## 파일 업로드 취약점

- **업로드 파일의 크기와 종류를 제한하지 않는 경우**
  ->(크기를 제한하지 않을 경우) 서버의 디스크 자원 또는 연결 자원을 고갈 시킨다. => 서비스 거부 공격 (DoS 공격)
  ->(종류를 제한하지 않을 경우) 서버에서 실행될 수 있는 파일(SSS,웹쉘)을 업로드해서 실행하여 서버의 제어권을 탈취
  -> 클라이언트에서 실행될 수 있는 악성 코드가 포함된 파일을 업로드 할 경우, => 악성코드의 유포지로 악용될 수 있다.
- **업로드 한 파일을 외부에서 접근 가능한 경로에 저장하는 경우**
  -> WebRoot 아래 경로에 파일이 저장되는 경우

### 방어기법

1) 업로드 파일의 크기를 제한한다.
2) 업로드 파일의 종류를 제한한다.
	a) 확장자를 비교
	b) Content-Type을 비교
	c) File Signature 
3) 업로드 파일을 외부에서 접근할 수 없는 경로에 저장한다.
	=> WebRoot 밖에 저장한다.
4) 업로드 파일의 저장 경로와 파일명을 외부에서 알 수 없도록 변경해서 사용한다.
	=> 파일명을 랜덤값, 날짜시간, 일련번호 등
5) 업로드 파일의 실행 속성을 제거하고 저장한다.



## 파일 다운로드 취약점

- 경로 조작 여부를 검증하지 않거나 다운로드 파일의 내용을 검사하지 않고 다운로드 기능을 제공하는 경우에 발생하게 된다.
  => 권한 밖의 디렉토리 또는 파일에 접근이 가능해진다.
  => 악성 코드가 포함된 파일이 다운로드 될 수 있다.

### 방어기법

1) 경로조작 문자열 포함 여부를 확인
2) 다운로드 전 파일의 무결성 또는 내용을 검증
