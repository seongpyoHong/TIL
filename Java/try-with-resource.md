### try-with-resource

전형적인 try-catch-finally 구문을 사용한 Stream 처리 방법은 다음과 같다.

```java
try {
  InputStream in = connection.getInputStream(); 
  OutputStream out = connection.getOutputStream();
  
  ... 
  ...
    
} catch (IOException e) {
  log.error(e.getMessage());
} finally {
  if(out != null) {
    try { 
      out.close();
    } catch (IOException e) { 
      e.printStackTrace(); 
    } 
  }
}
```

finally 구문은 try 구문에서 에러가 발생할 경우, 열린 stream을 닫아 Memory Leak을 방지한다. 하지만, 위와 같은 방법은 실제 개발자가 작성해야 할 코드 보다 Stream에 대한 예외 처리를 위한 코드가 더 많은 부분을 차지한다. 



Java 7부터 try-with-resource 문법을 적용해 자원을 반납하는 방법이 가능해졌다.

```java
try (InputStream in = connection.getInputStream(); OutputStream out = connection.getOutputStream()) {
  HttpRequest httpRequest = new HttpRequest(in);
  HttpResponse httpResponse = new HttpResponse(out);

  ...
  ...
    
} catch (IOException e) {
  log.error(e.getMessage());
}
```

try 조건에서 에러가 발생하는 경우 자동으로 열었던 stream을 닫아주기 때문에 기존 코드에서 finally 부분을 작성하며 발생했던 불필요한 코드를 줄일 수 있다.