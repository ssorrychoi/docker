## Fild Upload 취약점

BoardController.jsp

```java
//소스코드 분석
//	-외부에서 접근 가능한 경로에 업로드 파일을 저장하고 있음.
//	-업로드 파일의 이름을 그대로 사용
//	-파일의 크기와 종류를 제한하지 않고 있음
//	==> "파일 업로드 취약점이 존재한다"라고 한다.

//대응방안
//	1) 파일의 크기를 제한한다.
//	2) 파일의 종류를 제한한다.
//	3) 저장되는 파일의 이름을 외부에서 유추할 수 없도록 변경한다.
//	4) 외부에서 접근할 수 없는 경로에 파일을 저장한다.

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
```



### 방어기법

BoardController.jsp

```java
// 	파일 저장 경로를 상수로 정의
final static String UPLOAD_FILE_PATH = "c:\\\\upload\\files\\";

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

  //  소스 코드 분석
  // 	- 외부에서 접근 가능한 경로에 업로드 파일을 저장하고 있음
  //  - 업로드 파일의 이름을 그대로 사용하고 있음
  //  - 파일의 크기와 종류를 제한하지 않고 있음
  //  ==> 파일 업로드 취약점이 존재

  //  대응 방안
  //  1. 파일의 크기를 제한
  //  2. 파일의 종류를 제한
  //  3. 저장되는 파일의 이름을 외부에서 유추할 수 없도록 변경
  //  4. 외부에서 접근할 수 없는 경로에 파일을 저장

  // #1 파일 저장 경로를 외부에서 접근할 수 없는 경로로 변경
  //    WebRoot 밖으로 변경
  String uploadPath = UPLOAD_FILE_PATH;
  File dir = new File(uploadPath);
  if (!dir.exists()) {
    dir.mkdir();
  }

  MultipartFile file = request.getFile("file");

  // #2 업로드 파일의 크기를 제한
  //    파일의 크기가 2MB 이상이면 list.do로 리다이렉트
  if (file != null && file.getSize() >= 2048000) {
    return "redirect:list.do";
  }


  if (file != null && !"".equals(file.getOriginalFilename())) {
    String fileName = file.getOriginalFilename();

    // #3 파일의 확장자 검증을 수행
    //    png, jpg 이외의 확장자를 가진 파일은 저장하지 않도록 처리
    if (fileName.endsWith(".png") || fileName.endsWith(".jpg")) {

      // #4 파일 저장에 사용하는 이름을 외부에서 알 수 없도록 변경
      //    난수화해서 저장
      String savedFileName = UUID.randomUUID().toString();

      // File uploadFile = new File(uploadPath + fileName);
      File uploadFile = new File(uploadPath + savedFileName);
      /*	난수를 이용해서 파일을 저장하므로 
				 *	중복 파일이 생성되지 않으므로 해당 로직을 생략
				if (uploadFile.exists()) {
					fileName = new Date().getTime() + fileName;
					uploadFile = new File(uploadPath + fileName);
				}
				*/

      try {
        file.transferTo(uploadFile);
      } catch (Exception e) {
        System.out.println("upload error");
      }

      // 파일 업로드에 성공한 경우에만 
      // 파일 정보를 DB에 저장하기 위해서 값을 설정
      boardModel.setFileName(fileName); // 원본 파일명
      boardModel.setSavedFileName(savedFileName); // 저장에 사용한 파일명 

    } 
  }
```



### File Download

BoardController.jsp

```java
// File Download
@RequestMapping("/get_image.do")
public void getImage(HttpServletRequest request, HttpSession session, HttpServletResponse response){
  //	파일 정보를 추출하기 위해서 게시물 번호를 요청 파라미터에서 추출
  //	 = /get_image.do?idx=게시판번호

  // idx 파라미터가 없거나 또는 잘못 전달되는 경우에는 반환
  int idx=0;
  try{
    idx = Integer.parseInt(request.getParameter("idx"));
  }catch(Exception e){
    return;
  }

  //	idx를 이용해서 게시판 정보를 조회
  BoardModel board = service.getOneArticle(idx);

  //	저장된 파일명과 원본 파일명 추출
  String fileName = board.getFileName();
  String savedFileName = board.getSavedFileName();

  //	파일 저장 경로를 설정
  String filePath = UPLOAD_FILE_PATH + savedFileName;

  //	저장된 파일을 읽어서 응답을 통해서 브라우저로 전달
  BufferedOutputStream out = null;
  InputStream in = null;

  try{
    response.setContentType("image/jpeg");
    response.setHeader("Content-Disposition", "inline;filename="+fileName);

    File file = new File(filePath);
    in = new FileInputStream(file);

    out = new BufferedOutputStream(response.getOutputStream());
    int len; //파일에서 읽어들인 데이터의 길이
    byte[] buf = new byte[1024];	//파일에서 읽어들인 데이터를 저장 공간
    while( (len=in.read(buf)) > 0){
      out.write(buf,0,len);
    }

  }catch (Exception e){

  }finally{
    //자원 해제 구문
    if(out!=null){
      try{out.close();}catch (Exception e){}
    }
    if(in!=null){
      try{in.close();}catch (Exception e){}
    }
  }
}
```

view.jsp

```jsp
<th colspan="4">내용</th>
</tr>
<c:if test="${board.fileName != null}">
  <tr>
    <td colspan="4" align="left">
      <!-- 업로드 파일을 직접 참조(호출)하는 방식 -->
      첨부파일 : 
      <br /><a href="../files/${board.fileName}" target="_blank">${board.fileName}</a>
      <br /><img src="../files/${board.fileName}" />
      <br/>
      <!--파일 다운로드 기능을 이용해서 다운로드하는 방식  -->
      <br /><a href="get_image.do?idx=${board.idx}" target="_blank">${board.fileName}</a>
      <br /><img src="get_image.do?idx=${board.idx}" />
    </td>
  </tr>
</c:if>
```



## 중요 정보 노출

- 데이터 처리 중 중요 정보(주민번호, 패스워드, 카드번호 등)가 적절하게 보호되지 못하는 경우 개인정보 유출과 같은 침해 사고가 발생하게 된다.
- 이를 보완하기 위해 데이터 저장시 안전한 암호화 알고리즘을 적용해야 하며, 데이터 전송시에도 SSL 등의 안전한 통신 채널을 이용하거나 안전한 암호화 알고리즘에 의해 암호화된 데이터를 전송하도록 해야한다.



<로그인>

인증시도 횟수제한

세션

패스워드 해쉬화해서 저장

패스워드 전송단계에서 보호되어야한다. =>로그인페이지는 HTTPS해야한다.

<게시판>

XSS -> DB에 스크립트코드가 저장될 수 있다.

파일 업로드 / 다운로드 -> 파일의 종류,크기제한해야한다.

CSRF공격이 가능하다.





