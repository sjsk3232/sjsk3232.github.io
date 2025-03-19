---
title: 자바의 Reflection과 Annotation
date: 2025-03-11 21:43:25 +0900
categories: [Language, Java]
tags: [java, reflection, 리플렉션, annotation, 어노테이션]
math: true
---

## **리플렉션 (Reflection)**  
리플렉션은 런타임에 클래스/인터페이스의 메타 정보를 동적으로 가져오거나 수정하는 것을 말한다.

> **메타 정보 (Metadata)란?**  
> 패키지, 타입, 멤버(필드, 생성자, 메소드) 정보 등을 말한다.  
> 클래스로더가 바이트 코드(.class)를 읽고 메소드 영역에 클래스의 메타 데이터를 저장한다.
{: .prompt-tip}

<br>

### **1. Reflection의 사용**

<br>

#### **1-1. Class 객체 얻기**
특정 클래스의 메타 정보에 접근하기 전에 해당 클래스의 Class 객체를 얻어야 한다.

```java
Class targetClass = 클래스이름.class;
Class targetClass = 객체.getClass();
Class targetClass = Class.forName("패키지1.패키지2...클래스이름");
```

<br>

#### **1-2. 메타 정보 접근**
위에서 얻은 class 객체를 통해 클래스의 필드, 메소드, 생성자 등 여러 메타 정보에 접근 가능하다.

- 패키지, 타입(클래스, 인터페이스) 정보 얻기

```java
String packageName = targetClass.getPackage().getName(); // 패키지 이름
String classSimpleName = targetClass.getSimpleName(); // 패키지를 제외한 타입 이름
String classfullName = targetClass.getName(); // 패키지를 포함한 전체 타입 이름
```

- 필드, 생성자, 메소드 정보 얻기

```java
Field[] fields = targetClass.getDeclaredFields();  // 필드 정보 얻기
Method[] methods = targetClass.getDeclaredMethods();  // 메소드 정보 얻기
Constructor[] constructors = targetClass.getDeclaredConstructors();  // 생성자 정보 얻기
```

- 어노테이션 정보 얻기
```java
MyAnnotation annotation = method.getDeclaredAnnotation(MyAnnotation.class);
String value = annotation.value();
```

- 필드 읽기/쓰기  
설령 필드의 접근 제어자가 private여도 접근 가능하다.

```java
class Person {
    private String name = "John";
}

Person person = new Person();
Field field = person.getClass().getDeclaredField("name");

field.setAccessible(true);  // private 필드 접근 허용
System.out.println(field.get(person));  // "John" 출력

field.set(person, "Bob");  // 값 변경
System.out.println(field.get(person));  // "Bob" 출력
```

- 메소드 동적 호출  
필드와 마찬가지로 메소드도 접근 제어자가 private여도 접근 가능하다.

```java
class Example {
    private void privateMethod() {
        System.out.println("Private Method");
    }
}

Example example = new Example();
Method method = example.getClass().getDeclaredMethod("privateMethod");
method.setAccessible(true);  // private 접근 가능하도록 설정
method.invoke(example);  // "Private Method" 출력
```

이외에도 리소스 경로 얻기, 생성자로 객체 생성 등 여러가지 동작을 수행할 수 있다.

---
## **어노테이션 (Annotation)**  
어노테이션이라는 단어의 의미는 `주석`이다.  
하지만, 일반적인 주석과는 다르게 코드에 메타 데이터와 동작을 추가할 수 있다.  

<br>

### **1. Built-in Annotation**
자바에서 기본적으로 제공해주는 어노테이션이다.  
다음은 자주 사용되는 빌트인 어노테이션들이다.

<br>

#### **1-1. @Override**
메소드를 오버라이드할 때 메소드 선언 앞에 붙여주는 어노테이션이다.  
부모 클래스나 인터페이스에 해당 메소드와 동일한 시그니처의 메소드가 선언되어 있지 않다면 컴파일 오류를 발생시킨다.

<br>

#### **1-2. @Deprecated**
Deprecated된 클래스, 메소드나 필드 앞에 붙이는 어노테이션이다.  
해당 어노테이션이 붙은 요소를 사용하려고 할 때 경고해준다.  

<br>

#### **1-3. @SuppressWarnings**
Deprecated된 메소드를 사용하는 것이 권장되지는 않음을 인지하고 있지만 사용해야 할 때와 같이 경고를 출력하지 않게 하기 위해 사용되는 어노테이션이다.

```java
@SuppressWarnings("all") // 모든 경고 무시
@SuppressWarnings("cast") // 타입 캐스트 관련 경고 무시
@SuppressWarnings("deprecation") // Deprecated 요소 사용 경고 무시
@SuppressWarnings("null") // null 관련 경고 무시
...
```

<br>

### **2. Meta Annotation**
다른 어노테이션에 사용되는 어노테이션으로, 주로 커스텀 어노테이션을 생성할 때 사용된다.  

<br>

#### **2-1. @Retention**
어노테이션의 유지 기간을 설정할 때 사용하는 어노테이션이다.  

```java
@Retention(RetentionPolicy.RUNTIME) // 런타임에 적용, Reflection으로 접근 가능
@Retention(RetentionPolicy.Class) // JVM 메모리로 로딩할 때 적용
@Retention(RetentionPolicy.Source) // 컴파일할 때 적용
```

<br>

#### **2-2. @Target**
어노테이션의 적용 대상을 지정할 때 사용하는 어노테이션이다.  
적용 대상의 종류는 ElementType 열거 상수로 정의되어 있다.

| ElementType 열거 상수 |           적용 요소           |
| :-------------------: | :---------------------------: |
|         TPYE          | 클래스, 인터페이스, 열거 타입 |
|    ANNOTATION_TYPE    |          어노테이션           |
|         FIELD         |             필드              |
|      CONSTRUCTOR      |            생성자             |
|        METHOD         |            메소드             |
|    LOCAL_VARIABLE     |           로컬 변수           |
|        PACKAGE        |            패키지             |

```java
@Target({ElementType.TYPE, ElementType.FIELD, ElementType.METHOD}) // 적용 대상은 클래스, 필드, 메소드
```

<br>

#### **2-3. @Inherited**
부모 클래스의 어노테이션을 자식 클래스가 상속받도록 설정하는 어노테이션이다.

이외에도 같은 어노테이션을 여러 번 적용 가능하도록 설정하는 @Repeatable 어노테이션 등이 있다.

<br>

### **3. Custom Annotation**
위의 메타 어노테이션을 사용해서 사용자 정의 어노테이션을 만들 수 있다.

```java
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
        String value() default "myAnnotation";
}
```
