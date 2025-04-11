---
title: 커넥션풀과 DataSource
date: 2025-04-09 13:46:06 +0900
categories: [Backend, Spring]
tags: [Spring, 커넥션풀]
math: true
incomplete: false
---

## **커넥션풀**
DB와의 커넥션을 재사용할 수 있도록 관리되는 커넥션의 캐시이다.

<br>

### **커넥션풀 등장 배경**

![](/imgs/ConnectionPool_1.png)

커넥션을 획득하기 위해서는 위와 같이 복잡한 과정을 거쳐야 한다.  
때문에, 클라이언트의 요청이 있을 때마다 커넥션을 획득하게 되면 DB와 서버 모두 많은 리소스를 사용해야 한다.

이런 문제를 해결하기 위해, 커넥션을 미리 생성해두고 사용하는 커넥션 풀이라는 방법이 고안되었다.  

<br>

### **커넥션풀 동작 과정**

<br>

#### **1. 커넥션풀 초기화**

![](/imgs/ConnectionPool_2.png){: width="600" .normal}

서버가 시작될 때, 필요한 만큼의 커넥션을 미리 확보해서 커넥션풀에 보관한다.  
커넥션 수의 기본값은 보통 10개이다.  

<br>

#### **2. 커넥션 사용**

![](/imgs/ConnectionPool_3.png){: width="600" .normal}

커넥션풀에 커넥션을 요청하면, 커넥션풀이 idle 상태인 커넥션이 있다면 반환해준다.  
idle 상태인 커넥션이 없으면, 다른 커넥션이 다시 커넥션풀에 반납될 때까지 대기한다.  

<br>

#### **3. 커넥션 반납**

![](/imgs/ConnectionPool_4.png){: width="600" .normal}

커넥션풀에서 받은 커넥션으로 DB에 SQL을 전달하고 결과를 받아 로직을 처리한다.  
로직 처리가 끝나면 커넥션을 종료하지 않고, 커넥션풀에 해당 커넥션을 반납해준다.  

<br>

---
## **DataSource**

이전에 [JDBC 포스트](/posts/jdbc)에서 JDBC `DriverManager`으로 직접 커넥션을 획득하기 위해 `getConnection()` 메소드를 직접 구현했었다.  

하지만, `DBCP2`나 `HikariCP` 등의 커넥션풀을 사용해 커넥션을 획득하게 하려면, 서버 코드를 변경해야 한다.  
즉, OCP를 위반하게 된다.  

이를 방지하기 위해 JDBC에서 커넥션을 획득하는 방법을 추상화한 `DataSource`라는 인터페이스를 제공한다.

```java
public interface DataSource {
    Connection getConnection() throws SQLException;
}
```

![](/imgs/ConnectionPool_5.png)

위와 같이 대부분의 커넥션풀은 DataSource 인터페이스를 구현해놨다.  
또한, 우리가 이전에 직접 사용했던 `DriverManager`도 `DataSource`를 통해 사용할 수 있도록 `DriverManagerDataSource`라는 클래스를 제공한다.  

<br>

### **DataSource 사용**

이제 `DataSource`를 통해 커넥션을 획득해보자.  

```java
@RequiredArgsConstructor
public class MemberRepository {

    private final DataSource dataSource;

    public Member save(Member member) throws SQLException {
        String sql = "insert into member(member_id, age) values (?, ?)";

        Connection conn = null;
        PreparedStatement pstmt = null;

        try {
            conn = dataSource.getConnection();
            pstmt = conn.prepareStatement(sql);
            pstmt.setString(1, member.getMemberId());
            pstmt.setInt(2, member.getAge());
            pstmt.executeUpdate();
            return member;
        } catch (SQLException e) {
            throw e;
        } finally {
            close(conn, pstmt, null);
        }

    }

    // findById() {...}
    // update() {...}
    // delete() {...}

    private void close(Connection conn, Statement stmt, ResultSet rs) {
        JdbcUtils.closeResultSet(rs);
        JdbcUtils.closeStatement(stmt);
        JdbcUtils.closeConnection(conn);
    }
}
```

위 코드의 `save()`메소드를 살펴보면, 외부에서 주입받은 `DataSource`에서 `getConnection()` 메소드를 호출해서 커넥션을 얻고 사용할 수 있다.  

Config 클래스에서 HikariCP를 사용하도록 설정해주면, 스프링에서 `DataSource`로 주입해준다.  

> **커넥션풀의 `getConnection()`**  
> HikariCP와 같은 커넥션 풀을 사용할 때, `getConnection()` 메소드를 호출하면, `Connection` 인터페이스를 상속받은 `ProxyConnection`을 반환한다.  
> 그리고, `ProxyConnection`의 `close()` 메소드는 커넥션을 끊는 게 아닌, 커넥션 풀에 반환하는 식으로 오버라이드 되어 있다.  
> ```java
>public final void close() throws SQLException {
>    ...
>
>    try {
>        // AutoCommit이 설정되지 않았고, 변경사항이 발생했다면, rollback
>    } catch (SQLException e) {
>        // 예외 throw
>    } finally {
>        ...
>        this.poolEntry.recycle(this.lastAccess); // 커넥션풀에 반환
>    }
>}
> ```
{: .prompt-tip}
