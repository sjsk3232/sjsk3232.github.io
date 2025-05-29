---
title: 스프링 빈의 생명주기
date: 2025-05-28 22:31:24 +0900
categories: [Backend, Spring]
tags: [Spring, bean]
math: true
---

## **빈의 생명주기**
이전에 [스프링 컨테이너와 빈](/posts/spring-container) 포스트에서 빈에 대해 간단히 살펴봤다.  

빈의 생명 주기는 스코프마다 다른데, 주로 사용되는 싱글톤 스코프의 빈을 예시로 들어보면 다음과 같다.  

1) 스프링 컨테이너 생성  
2) 스프링 빈 객체 생성 및 컨테이너에 등록  
3) 의존관계 주입  
4) 초기화 콜백 호출  
5) 빈 사용  
6) 소멸 전 콜백 호출  
7) 스프링 종료  

> **객체 생성과 초기화의 분리**  
> 생성자에서는 객체의 생성(메모리 할당)과 동시에 파라미터를 받아서 필드를 초기화시킬 수 있다.  
> 필드에 값만 대입하는 단순한 초기화 과정일 경우는 객체 생성과 초기화를 한 번에 진행하는 것이 좋을 수 있다.  
> 하지만, 외부 커넥션을 연결하는 등의 무거운 동작을 수행하는 경우는 객체의 생성과 초기화를 나누는 것이 유지보수에 유리하다.
{: .prompt-tip}

### **빈 생명주기 콜백**
빈의 초기화, 소멸 전 콜백을 설정하는 방법은 크게 세 가지 방법이 있다.  

#### **1. 인터페이스 InitializingBean, DisposableBean의 사용**
스프링 초창기에 나온 방법이고, 더 좋은 방법이 있어서 거의 사용하지 않는다.  

인터페이스 InitializingBean에는 `afterPropertiesSet()` 메소드가, DisposableBean에는 `destroy()` 메소드가 선언되어 있다.  

`afterPropertiesSet()` 메소드를 오버라이드하면 초기화 콜백을 지원하고, `destroy()` 메소드를 오버라이드하면 소멸 전 콜백을 지원한다.  

```java
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;

public class ClientBean implements InitializingBean, DisposableBean {
    ...

    @Override
    public void afterPropertiesSet() throws Exception {
        connect(); // 초기화 동작 구현
    }
    
    @Override
    public void destroy() throws Exception {
        disConnect(); // 소멸 전 동작 구현
    }
}
```

> **단점**  
> - 스프링 전용 인터페이스에 의존하게 된다.  
> - 인터페이스에 선언된 메소드를 오버라이드하기 때문에, 초기화와 소멸 메소드의 이름 변경이 어렵다.  
> - 외부 라이브러리 등 직접 코드를 수정할 수 없는 경우는 적용할 수 없다.  
{: .prompt-warning}

#### **2. @Bean 어노테이션의 속성 설정**
`@Bean` 어노테이션을 사용할 때, `initMethod`와 `destroyMethod` 속성에 각각 초기화, 소멸 전 콜백할 메소드의 이름을 설정하는 방법이 있다.

예를 들면, `ClientBean` 클래스에 `init()`, `close()` 라는 이름의 메소드가 있고, 각각 초기화, 소멸 전 콜백되게 하고 싶으면 다음과 같이 설정할 수 있다.  

```java
@Configuration
public class ClientConfig {

    @Bean(initMethod = "init", destroyMethod = "close")
    public ClientBean clientBean() {
        ClientBean clientBean = new ClientBean();
        clientBean.setUrl("http://tmp.dev");
        return clientBean
    }
}
```

- 콜백할 메소드의 이름을 자유롭게 설정할 수 있다.  
- 빈으로 등록할 클래스의 코드를 스프링에 의존적이지 않게 작성할 수 있다.  
- 외부 라이브러리 등 직접 코드를 수정할 수 없는 경우에도 적용 가능하다.  

> **종료 메소드 추론**  
> 라이브러리는 대부분 `close()`, `shutdown()` 이라는 이름의 종료 메소드를 사용한다.  
> `@Bean`의 `destroyMethod`는 기본값이 `(inferred)`로 설정되어 있는데, inferred는 추론이라는 의미이다.  
> 이 추론 기능은 `close()`, `shutdown()` 이라는 이름의 메소드를 자동으로 호출해준다.  
> 따라서, 종료 메소드의 이름이 `close()`라면, 따로 `destroyMethod`를 설정해주지 않아도 된다.  
> 종료 메소드를 호출하고 싶지 않아서 추론 기능을 사용하기 싫으면, `destroyMethod=""`와 같이 공백으로 설정하면 된다.  
{: .prompt-info}

#### **3. @PostConstruct, @PreDestroy 어노테이션 사용**

초기화, 소멸 전 콜백할 메소드 위에 어노테이션을 붙여서 사용한다.  

```java
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

public class NetworkClient {
    ...
    
    @PostConstruct
    public void init() {
        connect();
    }
    
    @PreDestroy
    public void close() {
        disConnect();
    }
}
```

자바 표준에서 제공하는 기능이므로, 스프링에 종속적이지 않다.  

스프링에서도 가장 권장하는 방법이다.

다만, 클래스 내부에 어노테이션을 추가하는 방식이므로, 외부 라이브러리 등 직접 코드를 수정할 수 없는 경우는 적용할 수 없다.  

따라서, 기본적으로는 `@PostConstruct`, `@PreDestroy` 어노테이션을 사용하되, 코드를 고칠 수 없는 외부 라이브러리를 초기화, 종료해야 할 때는 `@Bean` 어노테이션의 속성을 설정하도록 하자.
