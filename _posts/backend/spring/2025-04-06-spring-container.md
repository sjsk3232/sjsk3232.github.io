---
title: 스프링 컨테이너와 빈
date: 2025-04-06 21:31:24 +0900
categories: [Backend, Spring]
tags: [Spring, spring container, bean]
math: true
---

## **스프링 빈**
스프링 컨테이너가 관리하는 재사용 가능한 컴포넌트(자바 객체)를 뜻한다.  

### **BeanDefinition**
BeanDefinition은 빈 실정 메타정보를 저장하는 인터페이스이다.  
BeanDefinition의 속성에 따라 스프링 컨테이너가 빈을 어떻게 생성하고 관리할지 결정된다.  

BeanDefinition에 있는 정보는 다음과 같다.  
- BeanClassName : 생성할 빈의 클래스 명
- factoryBeanName : 팩토리 역할의 빈을 사용할 경우의 이름, ex) appConfig
- factoryMethodName : 빈을 생성할 팩토리 메소드 지정, ex) memberService
- Scope : 빈이 존재할 수 있는 범위  
  - 싱글톤 : 디폴스 스코프, 스프링 컨테이너의 시작 ~ 종료까지 유지된다.  
  - 프로토타입 : 스프링 컨테이너가 프로토타입 빈의 생성 및 의존 관계 주입까지만 유지하고, 더 관리하지 않는다.  
  - 웹 관련 스코프 : 이외에도 request, session 등 여러 웹 관련 스코프가 존재한다.  
- lazyInit: 스프링 컨테이너를 생성할 때 빈을 생성하는 것이 아니라, 실제 빈을 사용할 때까지 최대한 생성을 지연 처리하는지 여부
- InitMethodName: 빈을 생성하고, 의존관계를 적용한 뒤에 호출되는 초기화 메소드 명
- DestroyMethodName: 빈의 생명주기가 끝나서 제거하기 직전에 호출되는 메소드 명
- Constructor arguments, Properties: 의존관계 주입에서 사용한다. (자바 설정처럼 팩토리 역할의 빈을 사용하면 없음)

<br>

---
## **스프링 컨테이너**
이전에 [IoC와 DI](/posts/ioc-di)에 대해서 살펴볼 때, `DI 컨테이너`에 대해 알아봤었다.  
스프링에서는 `스프링 컨테이너`라고 하는 DI 컨테이너를 제공한다.  

<br>

### **스프링 컨테이너란?**
객체(빈)를 생성 및 관리하고, 객체(빈) 간의 의존관계를 주입하는 DI 컨테이너이다.  

<br>

### **스프링 컨테이너 인터페이스**

![](/imgs/SpringContainer_1.png){: width="170" .normal}

#### **1. BeanFactory**
스프링 컨테이너의 최상위 인터페이스이다.  
빈을 조회하는 `getBean()` 메소드 등의 다양한 추상 메소드를 제공한다.  

#### **2. ApplicationContext**
BeanFactory를 상속받은 인터페이스이다.  
보통 스프링 컨테이너라고 하면, ApplicationContext를 말하는 것이다.  
BeanFactory 외에도 몇몇 인터페이스를 상속받기 때문에, 지역에 따라 다른 언어로 출력하는 국제화 기능이나 파일, 클래스 경로 등 리소스 조회 등 여러 기능에 대한 추상 메소드를 제공한다.

#### **3. AnnotationConfigApplicationContext**
ApplicationContext 인터페이스를 구현한 구현체이다.  

![](/imgs/SpringContainer_2.png){: width="500" .normal}

스프링 컨테이너는 위와 같이 자바 코드, XML 등 다양한 형식의 설정 정보를 받아들일 수 있도록 설계되었다.  

> **다양한 설정 형식 지원 방법**  
> ![](/imgs/SpringContainer_3.png){: width="500" .normal}  
> `ApplicationContext`의 구현체에서 설정 형식에 따라 각각 다른 `BeanDefinitionReader`를 사용해서 설정 정보를 읽고 `BeanDefinition`을 생성한다.  
> 그 후, BeanDefinition을 기준으로 빈을 생성 및 관리한다.
{: .prompt-info}

```java
@Configuration
class AppConfig {
  @Bean
  public MemberRepository memberRepository() {
    return new MemoryMemberRepository();
  }
}

ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
```

AnnotationConfigApplicationContext은 위와 같이 빈들의 설정 정보가 어노테이션 기반의 자바 코드일 경우에 사용할 수 있다.

<br>

### **싱글톤 컨테이너**
객체 인스턴스를 싱글톤 방식으로 관리하는 컨테이너를 싱글톤 컨테이너라고 한다.  
스프링 컨테이너는 기본적으로 빈을 싱글톤 방식으로 관리하기 때문에, 싱글톤 컨테이너 역할을 한다.  

> **싱글톤 방식의 주의점**  
> 싱글톤 방식은 하나의 객체(빈)을 공유하기 때문에, <mark style='background-color: #ffdce0'>무상태(stateless)로 설계해야 한다.</mark>  
> - 클라이언트가 값을 변경할 수 있는 등 클라이언트에 의존적인 필드를 사용하면 안된다.  
> - 필드 대신에 블럭 내부에서만 유지되는 지역 변수나 파라미터 등 공유되지 않는 것들을 사용해야 한다.  
> 
> 만약 공유필드를 사용하면, 특정 작업 도중 다른 쓰레드에 의해 값이 변경되어 의도와 다르게 동작하는 경우가 발생할 수 있고, 이는 논리적 오류이기 때문에 원인을 찾아내는게 매우 어려울 수 있다.
{: .prompt-warning}
