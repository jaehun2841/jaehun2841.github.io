---
title: Item 9. try-finally 보다는 try-with-resources를 사용하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item9
typora-copy-images-to: effective-java-item9
date: 2019-01-06 16:30:31
subtitle:
header-img:
---

# 서론
Java 라이브러리 중에는 close메서드를 호출해 직접 닫아줘야 하는 자원이 많다. InputStream, OutputStream, Connection등이 있다.  
자원 닫기는 클라이언트가 놓치기 쉽기 때문에 예측할 수 없는 성능 문제로 이어지기도 한다.  

나의 경우에도 신입 때 JDBC로 개발을 해야 했던 적이 있었는데,  
DB의 max-connection이 10이어서 11번째 호출 시 부터는 JDBCException이 발생하는 오류를 겪은 적이 있다.  
알고 보니 하나의 메서드에서 Connection 객체에 대한 참조를 해제 하지 않아 계속 Connection을 물고 있는 상황이었다.  
Spring을 사용하면서는 Mybatis나 JPA같은 프레임웍 내에서 자원에 대한 관리를 해주기 때문에 이런 일을 만날 일은 없게 되었다.

하지만 예전처럼 JDBC를 사용하거나, IO관련 작업을 할 때는 반드시 close를 통해 자원에 대한 해제가 필요하다.  
이전에는 finallizer를 통해 했다고는 아는데 이전 장에서 봤듯이 안 쓰는것이 좋다.  
그리고 try-catch-finally 구문을 통해 Exception이 발생하더라도, finally 절에서 close를 호출 하도록 했는데, 코드가 복잡해 지고, finally 절 내에서 Exception이 발생하는 경우도 있기 때문에 이를 방지하기 위해 더 더러운 코드를 생산하는 일이 적지 않았다.

옛날에 자주 사용해 본 아주아주 슬픈 코드이다.
```java
public class JDBCExam {
  Connection connection;
  Statement statement;
  ResultSet resultSet;

  String driverName = "oracle.jdbc.driver.OracleDriver";
  String url = "oracle:thin:localhost:1521:ORCL";
  String user = "scott";
  String password = "tiger";

  public JDBCExam() {
      try {
          Class.forName(driverName);
          connection = DriverManager.getConnection(url, user, password);
      } catch (ClassNotFoundException e) {
          System.out.println("[로드 오류]\n" + e.getStackTrace());
      } catch (SQLException e) {
          System.out.println("[연결 오류]\n" + e.getStackTrace());
      } finally {
          try {
              if (connection != null) {
                  connection.close();
              }
              if (statement != null) {
                  statement.close();
              }
              if (resultSet != null) {
                  resultSet.close();
              }
          } catch (SQLException e) {
              System.out.println("[닫기 오류]\n" + e.getStackTrace());
          }
      }
  }
}
```

# AutoCloseable
JDK 1.7 부터 try-with-resources 구문이 추가 되었고, `AutoCloseable` 인터페이스가 추가되었다. 이 인터페이스가 나오고 부터 Java의 close를 지원하는 클래스에서 AutoCloseable을 구현하도록 변경되었다.

# try-with-resources
try 절에 AutoCloseable을 구현한 클래스에 대한 인스턴스를 선언하게 되면 try절이 종료되는 시점에 AutoCloseable 인터페이스의 close() 메서드를 자동으로 호출 한다. 그렇기 때문에 이전처럼 finally 구문에서 별도로 자원에 대한 해제를 하지 않아도 된다.

위의 예시코드를 try-with-resources 구문으로 바꿔보았다.  
위의 코드보다는 한층 깔끔해 짐을 볼 수 있다.
```java
public class JDBCExam {

    String driverName = "oracle.jdbc.driver.OracleDriver";
    String url = "oracle:thin:localhost:1521:ORCL";
    String user = "scott";
    String password = "tiger";

    public JDBCExam() {
        try (Connection connection = DriverManager.getConnection(url, user, password);
             Statement statement = connection.prepareStatement("select * from member");
             ResultSet resultSet = statement.getResultSet();
             ){
            Class.forName(driverName);
            String name = resultSet.getString(1);
            System.out.println("회원명 : " + name);
            
        } catch (ClassNotFoundException e) {
            System.out.println("[로드 오류]\n" + e.getStackTrace());
        } catch (SQLException e) {
            System.out.println("[연결 오류]\n" + e.getStackTrace());
        } 
    }
}
```

# 참고
* Effective Java 3rd Edition - Item 9. try-finally 보다는 try-with-resources를 사용하라