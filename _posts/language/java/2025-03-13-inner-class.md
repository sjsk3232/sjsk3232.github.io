---
title: 내부 클래스란?
date: 2025-03-11 21:43:25 +0900
categories: [Language, Java]
tags: [java, inner class, 내부 클래스]
math: true
---

## **내부 클래스 (Inner class)**
클래스(Outer class)의 내부에 선언된 클래스를 의미한다.  
중첩 클래스라고도 한다.

> **참고**  
> ![](/imgs/inner_1.png){: .normal}  
> 위와 같이 컴파일로 생성되는 클래스 파일은 `외부클래스$내부클래스.class`와 같은 형식의 이름이 된다.  
{: .prompt-tip}

### **내부 클래스의 종류**

- Instance member class
- static member class
- local class
- anonymous class

#### **인스턴스 멤버 클래스 (Instance member class)**
`static` 키워드 없이, 클래스의 인스턴스 멤버로 선언된 클래스이다.

```java
class Outer {
    private static int outerPrice = 5000;
    private int outerSize = 1;

    private void outerPrint() {
        System.out.println("Outer");
    }

    public void outerToInner() {
        Inner inner = new Inner();

        // 인스턴스 클래스의 객체를 통해서 인스턴스 클래스의 인스턴스 필드에 접근 가능
        int innerSize = inner.innerSize;

        // 인스턴스 클래스의 static 필드에는 클래스를 통해 가능
        int innerPrice = Inner.innerPrice;
    }

    class Inner {
        public static int innerPrice = 10000;
        public int innerSize = 2;

        public void innerPrint() {
            System.out.println("Inner");
        }

        public void innerToOuter() {
            // 인스턴스 멤버 클래스에서는 outer 클래스의 필드가 private라도 직접 접근 가능
            int price = outerPrice;
            int size = outerSize;
            outerPrint();
        }
    }
}
```

내부 ⇨ 외부로 접근은 자유롭지만, 외부 ⇨ 내부는 직접 접근이 불가하다.  
외부 ⇨ 내부로 접근하려면 인스턴스 멤버 클래스의 객체를 통해 접근이 가능하다.

##### **인스턴스 멤버 클래스의 객체 생성**

```java
public class Main {

    public static void main(String[] args) {
        Outer outer = new Outer();
        Outer.Inner inner = outer.new Inner(); // 인스턴스 클래스 생성자 호출 방법
        
        // 아래와 같이 inner 클래스의 인스턴스를 생성할 수도 있다.
        Outer.Inner inner2 = new Outer().new Inner();

        outer = null // 객체 연결 끊음
        inner.showOuterSize(); // 1 출력
    }
}
```

암묵적으로 인스턴스 멤버 클래스의 객체는 외부 클래스의 인스턴스를 참조하고 있기 때문에, 위의 예시와 같이 **외부 클래스의 인스턴스가 먼저 생성되어야 인스턴스 멤버 클래스의 객체를 만들 수 있다.**

위 예제에서 outer 변수에 null을 할당해서 outer 객체와의 연결을 끊어도 inner 객체에서 outer 객체를 참조하고 있기 때문에 outer 객체가 GC에 의해 수거되지 않는다.

이 때문에 outer 변수에 null을 할당해도 inner 객체에서 outer 객체의 인스턴스 변수에 접근 가능한 것이고 이로 인해 **메모리 누수 현상이 발생할 수 있다. (가능하면 정적 멤버 클래스를 사용하라!)**

##### **동일한 이름의 외부 클래스 필드 및 메소드 접근 방법**
내부에서 외부 클래스의 필드와 메소드에 직접 접근 가능한 것은 위에서 살펴봤다.  
하지만 접근하려는 필드나 메소드의 이름이 내부 클래스에 이미 정의되어 있다면, 일반 클래스에서 `this` 키워드를 사용하듯이 `Outer.this`와 같이 `정규화된 this`를 통해 외부 클래스의 객체에 접근할 수 있다.  

```java
class Outer2 {
    private int size = 1;

    public void print() {
        System.out.println("Outer2");
    }

    class Inner2 {
        private int size = 5;

        public void print() {
            System.out.println(Outer2.this.price); // 5
            Outer2.this.print(); // Outer2
        }
    }
}
```

> 정규화된 this란?  
> 객체에서의 this가 객체 자신을 가리키는 것과 같이, 외부 클래스 객체를 가리키는 말로 `외부_클래스_이름.this`과 같은 형태로 나타낼 수 있다.  
> 유명한 서적인 이펙티브 자바에 등장하는 용어로 공식적으로 사용되는 용어는 아닌 것으로 알고 있다.
{: .prompt-info}

#### **정적 멤버 클래스 (Static member class)**
`static` 키워드가 붙은 내부 클래스이다.  
우리가 일반적으로 알고 있는 **static 필드, static 메소드는** static 영역에 저장되어 클래스 단위에서 공유되기 때문에, 클래스 로딩 시점에서 **단 한 번 메모리에 할당**되고 프로그램이 종료될 때까지 유지된다고 알고 있다.  
그래서 정적 멤버 클래스는 static 필드처럼 하나의 인스턴스만 존재해야 한다고 생각할 수도 있지만, 그렇지 않다.  
정적 멤버 클래스는 인스턴스 멤버 클래스와 다르게 **외부 클래스의 인스턴스 없이 독립적으로 존재**한다.  
때문에, 정적 멤버 클래스의 인스턴스는 여러 개 존재할 수 있다.  

```java
class Outer3 {
    private static String name = "Outer3";
    private int age = 10;
    private static void print() {
        System.out.println("Outer3");
    }
    
    static class Inner3 {
        public void print() {
            System.out.println(name); // 접근 가능
            // System.out.println(age); 접근 불가
            // System.out.println(Outer3.this.age); 정규화된 this 사용 불가
            Outer3.print(); // 동일한 이름의 멤버에 접근할 때는 클래스명을 통해 접근 가능
        }
    }
}
```

정적 멤버 클래스는 외부 클래스의 인스턴스를 참조하고 있지 않기 때문에, 위의 예시처럼 static 영역에 있는 name에는 직접 접근이 가능하지만 외부 클래스의 인스턴스 멤버인 age에는 직접 접근할 수 없다.  
또한, 외부 클래스의 인스턴스를 참조하고 있지 않기 때문에 정규화된 this도 사용이 불가하다.

##### **정적 멤버 클래스의 객체 생성**

```java
public class Main {

    public static void main(String[] args) {
        Outer3.Inner3 inner3_1 = new Outer3.Inner3();
        Outer3.Inner3 inner3_2 = new Outer3.Inner3();
        System.out.println(inner3_1 == inner3_2); // false
    }
}
```

인스턴스 멤버 클래스와는 다르게 **외부 클래스의 인스턴스 없이도 여러 개의 내부 클래스 인스턴스를 생성**할 수 있다.

#### **지역 클래스 (Local class)**
메소드, 생성자, 블록 내에 정의된 클래스이다.  
지역 클래스는 정의된 범위 내에서만 접근하는 지역 변수와 같은 개념이기 때문에 일반적인 클래스와 달리 JVM 스택 메모리에 적재돼고 정의된 범위를 벗어나면 GC에 의해 해제된다.  

```java
public class Main {

    public static void main(String[] args) {

        class Local {
            public void sayHello() {
                System.out.println("Hello World");
            }
        }

        Local local = new Local();
        local.sayHello();
    }
}
```

#### **익명 클래스 (Anonymous class)**
말 그대로 이름이 없는 클래스다.  
일반적으로 인터페이스나 추상 클래스를 구현하여 즉시 인스턴스 생성 후 사용할 때 주로 사용된다.  

```java
interface Greeting {
    void sayHello();
}

public class Main {
    public static void main(String[] args) {
        Greeting greeting = new Greeting() { // 익명 클래스 생성
            @Override
            public void sayHello() {
                System.out.println("Hello from Anonymous Class!");
            }

            public void print() {
                System.out.println("Anonymous!");
            }
        };

        greeting.sayHello();
        // greeting.print(); 사용 불가
    }
}
```

##### **새로 정의한 메소드 사용 불가**
주의할 점은 익명 클래스로 구현하고 생성한 객체의 타입은 부모 클래스나 인터페이스의 타입을 따르고, 부모 타입의 객체는 자식 클래스(익명 클래스)에서 별도로 추가한 메소드를 알 수 없기 때문에 위 예시의 print() 메소드처럼 호출이 불가하다.
