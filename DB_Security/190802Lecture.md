## 파라미터 조작과 잘못된 접근제어

- 파라미터 조작은 막을 방법이 없다. 
- 화면 / 기능 / 데이터 에서의 접근제어



WebGoat에서 실습

WebGoat > Access Control Flaws > LAB : Role Based Access Control Flaw Stage 1

ID : tom PW: tom 해서 들어가면 ViewProfile을 DeleteProfile로 바꿔서 전달하면 지워지게 된다.

❌ 접근 권한을 설정하지 않음.

따라서 접근 권한을 확인해야한다.

RoleBasedAccessControl.java

```java
public void handleRequest(WebSession s)
{
  // Here is where dispatching to the various action handlers happens.
  // It would be a good place verify authorization to use an action.

  // System.out.println("RoleBasedAccessControl.handleRequest()");
  if (s.getLessonSession(this) == null) s.openLessonSession(this);

  String requestedActionName = null;
  try
  {
    requestedActionName = s.getParser().getStringParameter("action");
  } catch (ParameterNotFoundException pnfe)
  {
    // Let them eat login page.
    requestedActionName = LOGIN_ACTION;
  }
  // System.out.println("Requested lesson action: " + requestedActionName);

  try
  {
    DefaultLessonAction action = (DefaultLessonAction) getAction(requestedActionName);
    if (action != null)
    {
      // System.out.println("RoleBasedAccessControl.handleRequest() dispatching to: " +
      // action.getActionName());
      if (!action.requiresAuthentication())
      {
        // Access to Login does not require authentication.
        action.handleRequest(s);
      }
      else
      {
        // ***************CODE HERE*************************

        // *************************************************

        // 	요청한 사용자가 인증받은 사용자인지를 확인
        if (action.isAuthenticated(s))
        {
          // 요청을 처리
          // 요청이 왔을 때 사용자의 인증 여부만 판단하고
          // 권한 여부를 판단하지 않고 요청을 처리
          // = 기능 레벨에서의 접근통제를 하지 않고 있다
          // action.handleRequest(s);

          // 조치방안
          // 요청한 사용자의 권한 여부를 확인 후 처리하도록 수정
          int employeeId = action.getUserId(s);
          String functionId = action.getActionName();
          if (action.isAuthorized(s, employeeId, functionId)) {
            action.handleRequest(s);
          } else {
            throw new UnauthorizedException();
          }
        }
        else
          throw new UnauthenticatedException();
      }
    }
```

