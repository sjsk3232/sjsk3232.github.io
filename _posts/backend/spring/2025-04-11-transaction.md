---
title: 트랜잭션과 트랜잭션 추상화
date: 2025-04-11 17:05:41 +0900
categories: [Backend, Spring]
tags: [Spring, 트랜잭션, ACID]
math: true
incomplete: false
---

## **트랜잭션**
DB 상태를 변화시키기 위해 수행되는 작업 단위

<br>

### **트랜잭션의 특성 ACID**
트랜잭션은 `원자성(Atomicity)`, `일관성(Consistency)`, `격리성(Isolation)`, `지속성(Durability)`을 보장해야 한다.  

<br>

#### **1. 원자성(Atomicity)**
트랜잭션 내에서 실행한 작업들은 마치 하나의 작업인 것처럼 모두 성공 하거나 모두 실패해야 한다.  

<br>

#### **2. 일관성(Consistency)**
모든 트랜잭션은 일관성 있는 데이터베이스 상태를 유지해야 한다.  
Not Null인 컬럼에 Null이 올 수 없는 것과 같이, DB에서 정한 무결성 제약 조건을 항상 만족하는 것을 예로 들 수 있다.  

<br>

#### **3. 격리성(Isolation)**
동시에 실행되는 트랜잭션들이 서로에게 영향을 미치지 않아야 한다.  
격리성은 동시성과 관련된 성능 이슈로 인해 트랜잭션 격리 수준(Isolation level)을 선택할 수 있다.  

> **트랜잭션 격리 수준(Isolation level)**  
> 1. **`READ UNCOMMITTED` (커밋되지 않은 읽기)**  
> 다른 트랜잭션이 변경했지만, 커밋하지 않은 데이터를 읽을 수 있다. (`Dirty Read`)  
> 2. **`READ COMMITTED` (커밋된 읽기)**  
> `Dirty Read`는 방지한다.  
> 같은 트랜잭션 내에서 특정 레코드를 여러 번 조회할 때, 다른 트랜잭션에 의해 해당 레코드의 값이 update되어, 조회 결과가 달라질 수 있다. (`Non-repeatable Read`)  
> 3. **`REPEATABLE READ` (반복 가능한 읽기)**  
> 특정 레코드에 Lock을 걸어서, `Non-repeatable Read`까지 방지한다.  
> 같은 트랜잭션 내에서 특정 조건에 해당하는 레코드들을 여러 번 조회할 때, 다른 트랜잭션에 의해 해당 조건에 맞는 레코드가 insert되어, 조회 결과가 달라질 수 있다. (`Phantom Read`)
> 4. **`SERIALIZABLE` (직렬화 가능)**  
> Range Lock을 걸어서, `Phantom Read`까지 방지한다.
{: .prompt-info}

<br>

#### **4. 지속성(Durability)**
트랜잭션을 성공적으로 끝내면 그 결과가 항상 DB에 정상적으로 반영되어야 한다.  
중간에 시스템에 문제가 발생해도 데이터베이스 로그 등을 사용해서 성공한 트랜잭션 내용을 복구해야 한다.

<br>

### **트랜잭션 과정**

![](/imgs/Transaction_1.png)

1. WAS에서 DB 드라이버를 통해 DB 서버에 연결을 요청한다.  
2. WAS와 DB 서버 간의 커넥션이 맺어지는 과정에서 DB 서버가 내부에 세션을 만든다.  
해당 커넥션을 통한 모든 요청은 이 세션을 통해서 실행된다.  
3. 커넥션에서의 요청으로 세션이 트랜잭션을 시작한다.  
4. 커밋 또는 롤백이 수행되면 트랜잭션이 종료된다.

<br>

### **트랜잭션 적용**
트랜잭션이 필요한 예시로 계좌이체를 들 수 있다.  
계좌이체 진행 과정 도중 실패해도, 한 계좌의 잔액만 늘거나 줄면 안되기 때문이다.  

따라서, 트랜잭션으로 비즈니스 로직(계좌이체 전체 과정)을 한 작업으로 묶어줄 필요가 있다.

스프링에서 계좌이체 과정에 트랜잭션을 적용해보자.  

![](/imgs/Transaction_2.png)  

```java
@Slf4j
@RequiredArgsConstructor
public class MemberService {

    private final DataSource dataSource;
    private final MemberRepository memberRepository;

    // 계좌 이체
    public void accountTransfer(String fromId, String toId, int money) throws SQLException {   
        // 1. 커넥션 생성
        Connection conn = dataSource.getConnection();
        try {
            conn.setAutoCommit(false); // 트랜잭션 시작
            // 2-1. 비즈니스 로직 수행 (update 두 번(출금, 입금) 수행)
            // 내부에서 리포지토리의 메소드를 
            bizLogic(fromId, toId, money, conn);
            // 3. 로직 성공 시 커밋
            conn.commit();
        } catch (Exception ex) {
            // 2-2. 로직 실패 시 롤백
            conn.rollback();
            throw new IllegalStateException(ex);
        } finally {
            // 4. 커넥션 반환
            releaseConnection(conn);
        }
    }
    ...

    private void releaseConnection(Connection conn) {
            if (conn != null) {
                try {
                    conn.setAutoCommit(true);  // 기본이 AutoCommit 모드이기 때문에, 커넥션 풀에 반납할 때는 true로 변경 필요
                    conn.close();
                } catch (Exception ex) {
                    log.info("error", ex);
                }
            }
     }
}
```

> **위 방식의 문제점**  
> 서비스 계층의 코드는 JDBC와 같은 특정 기술에 종속적이지 않아야 한다.  
> `DataSource`, `Connection` 등 JDBC에 의존하게 작성할 경우, JPA 등 다른 기술을 사용하려 한다면, 서비스 코드를 수정해야 하는 문제점이 있다.  
{: .prompt-warning}  

위 서비스 코드에서는 왜 `DataSource`나 `Connection`를 의존했을까?  

`DataSource`에서 얻은 `Connection`을 여러 비즈니스 로직 처리 메소드의 파라미터로 넘겨주는 등... 직접 트랜잭션 관련 로직을 처리하려고 했기 때문이다.

<br>

---
## **트랜잭션 매니저**
스프링의 트랜잭션 추상화 인터페이스 `PlatformTransactionManager`와 구현체를 말한다.  

<br>

### **트랜잭션 추상화**
JDBC나 JPA 등 트랜잭션을 사용하기 위해 작성하는 코드는 각각 다르기 때문에, 스프링에서는 트랜잭션의 시작, 커밋, 롤백과 같은 트랜잭션 기능을 추상화한 인터페이스를 제공해준다.  

```java
public interface PlatformTransactionManager extends TransactionManager {
      // 트랜잭션 시작 (이미 진행 중인 트랜잭션이 있으면 참여)
      TransactionStatus getTransaction(@Nullable TransactionDefinition definition)
              throws TransactionException;

      // 커밋        
      void commit(TransactionStatus status) throws TransactionException;
      
      // 롤백
      void rollback(TransactionStatus status) throws TransactionException;
}
```

![](/imgs/Transaction_3.png)

그리고 위 그림과 같이 스프링에서는 여러 기술에 해당하는 구현체도 제공해준다.  

<br>

### **트랜잭션 매니저의 동작 방식**
위에서 트랜잭션을 유지하기 위해 커넥션을 파라미터로 넘겨주는 방법을 사용했다.  

이와 다르게, 트랜잭션 매니저는 내부적으로 트랜잭션 동기화 매니저를 사용해서 커넥션을 보관해놓고, 꺼내 쓰는 방식으로 동작한다.  
트랜잭션 동기화 매니저는 `쓰레드 로컬`을 사용해 커넥션을 동기화해준다.

> **Thread Local**  
> 스프링에서 서비스나 리포지토리는 기본적으로 싱글톤 방식으로 관리되는 빈으로 등록된다.  
> 서비스나 리포지토리에 필드를 생성하게 되면, 여러 스레드가 해당 필드를 공유한다.  
> 
> 한 쓰레드만 접근 가능한 저장 공간이 필요한 경우를 위해 자바는 `쓰레드 로컬`을 제공한다.  
> `쓰레드 로컬`은 각각의 쓰레드에게 별도의 저장공간을 할당해서 해당 쓰레드만 접근 가능하게 해주는 Thread safe한 기술이다.  
> 
> `쓰레드 로컬`을 사용 후 정리하지 않으면 메모리적으로도 좋지 않을 뿐 아니라, 이후에 다른 클라이언트의 요청으로 해당 쓰레드를 사용할 때, 남아있던 정보가 응답으로 넘어갈 수 있다.
{: .prompt-info}

<br>

#### **1. 트랜잭션 시작**

![](/imgs/Transaction_4.png){: width="700" .normal}

1. 서비스에서 트랜잭션 매니저의 `getTransaction()` 메소드를 호출  
2. 트랜잭션 매니저가 내부의 `DataSource`를 사용해 커넥션 획득  
3. 획득한 커넥션을 수동 커밋 모드로 변경해 트랜잭션 시작  
4. 이후 커넥션을 트랜잭션 동기화 매니저에 보관  

<br>

#### **2. 로직 실행**

![](/imgs/Transaction_5.png){: width="700" .normal}

5. 서비스에서 비즈니스 로직을 처리하면서 리포지토리의 메소드들을 호출 (커넥션을 파라미터로 전달 X)
6. 리포지토리는 `DataSourceUtils`의 `getConnection()` 메소드를 사용해서 트랜잭션 동기화 매니저에 보관된 커넥션 획득 후 작업 진행

<br>

#### **3. 트랜잭션 종료**

![](/imgs/Transaction_6.png){: width="700" .normal}

7. 비즈니스 로직의 정상 완료 여부에 따라 트랜잭션 매니저의 `commit()`이나 `rollback()` 메소드 호출  
8. 트랜잭션 매니저가 트랜잭션 동기화 매니저에서 보관된 커넥션을 획득 후 커밋이나 롤백 진행  
9. 전체 리소스 정리
> - 트랜잭션 동기화 매니저 정리 (쓰레드 로컬 정리)  
> - 커넥션풀을 고려해서 커넥션을 자동 커밋 모드로 되돌림  
> - 커넥션의 `close()` 메소드를 호출해, 커넥션을 종료하거나 커넥션 풀에 반환한다.  

<br>

### **트랜잭션 매니저 적용**

```java
@RequiredArgsConstructor
public class MemberService {

    private final PlatformTransactionManager transactionManager;
    private final MemberRepository memberRepository;

    // 계좌 이체 
    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        // 1. 트랜잭션 시작
        TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());

        try {
            // 2-1. 비즈니스 로직 수행
            bizLogic(fromId, toId, money);
            
            // 3. 로직 성공 시 커밋
            transactionManager.commit(status);
        } catch (SQLException e) {
            // 2-2. 로직 실패 시 롤백
            transactionManager.rollback(status);
            throw new IllegalStateException(e);
        }
        
        // 4. 사용한 커넥션 반환은 트랜잭션 매니저가 처리
    }
    ...
}
```

트랜잭션 매니저를 적용하니, 서비스 코드가 JDBC와 같은 특정 기술에 의존하지 않게 되었다.  

그러나, 여전히 다음과 같은 문제가 남아있다.  

1. 성공시 commit, 실패시 rollback을 위한 try, catch, finally 코드가 반복된다.  
2. 예외 처리를 위해 JDBC 전용 예외인 SQLException가 사용됐다.  

이러한 문제들을 해결하기 위해, 트랜잭션 템플릿을 사용하고 트랜잭션 AOP를 적용하는 등 해야할 일이 많은데, 이 부분은 다음에 자세히 알아보도록 하자.
