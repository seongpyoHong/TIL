### Servlet

Controller, HttpRequest, HTTPResponse를 추상화해 인터페이스로 정의해 놓은 표준

- HTTP 요청 및 응답 헤더, 본문 처리에 시간을 투자하여 비즈니스 로직에 필요한 로직을 작성하는데 투자할 시간이 적어진다.



서버가 시작할 떄 서블릿 컨테이너는 서블릿 인스턴스를 생성해 요청 URL과 서블릿 인스턴스를 연결해 놓는다. 이후, 클라이언트에서 요청이 오면 요청 URL에 해당하는 서블릿 인스턴스를 찾아 보든 작업을 위임한다.

- 서블릿 컨테이너의 구현체는 Tomcat / Jetty / JBoss등이 존재한다.



서블릿 컨테이너는 중요 역할을 정리하면 다음과 같다. 

- 서블릿 클래스의 인스턴스 생성
- 요청 URL과 서블릿 인스턴스 매핑
- 클라이언트 요청에 해당하는 서블릿 컨테이너에 역할 위임
- 서블릿 인스턴스의 초기화 및 소멸 담당



**Servlet Instance**

```java
@WebServlet("/hello")
public class HelloWorldServlet extends HttpServlet {
  @Override
  protected void doGet(HttpServletRequest request, HttpServletResponse response) throws SelvletException, IOException{
    //Business Code
  }
}
```

- WebServlet 어노테이션의 값에 해당하는 URL로 요청이 올 경우 호출되는  HelloWorldServlet



**Servlet Interface**

```  java
public interface Servlet {
  public void init(ServletConfig config) throws ServletException;
  
  public void service(ServletRequest request, ServletResponse response);
  
  public void destroy();
  
  public SevletConfig getServletConfig();
  
  public String getServletInfo();
}
```



> Tip) Java 진영에서 `컨테이너` 라는 용어는 기본적으로 생명주기를 관리하는 기능을 제공함을 의미한다,  컨테이너가 관리하는 객체의 인스턴스는 사용자가 직접 인스턴스를 생성하는 것이 아니라 컨테이너가 관리하기 떄문에 만약 인스턴스의 생명주기를 관리하고자 할 경우, 초기화/소멸과 같은 메소드를 인터페이스로 만들어놓고 확장하여 사용한다. 
>
> ​		ex) Spring Bean Container



기본적으로 서블릿 컨테이너는 멀티스레드로 동작하여 동시에 여러 클라이언트가 접속할 수 있도록 지원한다. 이 때, 서블릿 컨테이너는 처음 시작할 때 모든 서블릿을 초기화하고 이를 모든 스레드가 같은 인스턴스를 재사용하는 방식으로 동작한다.