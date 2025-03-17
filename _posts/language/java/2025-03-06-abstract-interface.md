---
title: 추상 클래스와 인터페이스
date: 2025-03-06 19:24:16 +0900
categories: [Language, Java]
tags: [java, 상속, 다형성, 추상화, OOP, 객체 지향, 인터페이스]
math: true
---

## **추상 클래스**
추상 클래스는 추상 메소드를 포함할 수 있는 클래스를 말한다.  
일반적으로 추상 메소드를 포함시켜 추상 클래스를 정의하지만, 추상 메소드가 없을 수도 있다.  
java에 `Enum` 클래스가 없었을 때는 추상 클래스에 상수(public static)만 몰아넣어 `Enum` 클래스와 비슷하게 사용한 경우도 있다고 한다.  
혹은, 인스턴스를 생성시키고 싶지 않은 클래스를 정의할 때 사용하기도 한다.  
추상 메소드는 `abstract` 키워드를 사용하여 메소드의 선언부만 작성하고, 구현하지 않은 메소드이다.  
아래는 추상 클래스와 자식 클래스의 예시이다.  

```java
abstract class Animal {
    private String name;

    public Animal(String name) {
        this.name = name;
    }

    public abstract void move();
}

class Bird extends Animal {
    private int wingSize;

    public Bird(String name, int wingSize) {
        super(name);
        this.wingSize = wingSize;
    }
    
    public void move() {
        System.out.println("훨훨~");
    }
}
```

추상 클래스는 일반적인 클래스처럼 생성자, 멤버 변수, 메소드가 있지만, 추상 메소드와 같이 일부 추상적인 내용이 있기 때문에 직접 `new` 키워드를 사용해 객체를 생성할 수 없다.  
추상 클래스를 상속받은(`extends`) 자식 클래스를 통해 객체를 생성해야 한다.  

### **추상 클래스 사용 이유**

- 자식 클래스들의 공통된 필드 및 메소드를 통일된 방식으로 사용 가능 (**다형성**)
- 자식 클래스에게 추상 메소드에 대한 구현을 강제해 해당 메소드의 기능 동작을 보장
- 추상 클래스의 설계를 따라 구현에 집중 가능
- 유지보수, 확장성 증가

---
## **인터페이스**
인터페이스는 모든 메소드가 추상 메소드여야 하는 것은 아니지만 보통 추상 메소드의 집합이라고 정의한다.  
모든 필드는 상수로만 정의할 수 있기 때문에, 필드를 초기화할 때 `public static final`를 생략할 수 있다.  
또한 대부분의 메소드도 추상 메소드이기 때문에 추상 메소드를 선언할 때 `public abstract`를 생략할 수 있다.  
생략한 키워드들은 컴파일 시에 컴파일러가 추가해준다.  

```java
interface Animal {
    public abstract void move();
}

interface Wild {
    public abstract void attack();
}

class Bear implements Animal, Wild {
    public void move() {
        System.out.println("쿵! 쿵!");
    }
    
    public void attack() {
        System.out.println("뚜쉬!");
    }
}
```

인터페이스도 추상 클래스처럼 객체를 직접 생성할 수 없고, 인터페이스를 구현한(`implements`) 클래스로 객체를 생성할 수 있다.  
인터페이스의 최고 조상이 Object가 아니기 때문에, 최고 조상이 Object인 클래스를 상속받을 수 없고, 오직 인터페이스만 상속받을 수 있다.  
Java는 기본적으로 단일 상속만 지원하지만, 위 `Bear` 클래스처럼 여러 인터페이스를 다중으로 상속받을 수 있다.
> 인터페이스의 추상 메소드는 구현되어있지 않기 때문에, 다중 상속으로 인한 충돌이 발생하지 않기 때문

```java
interface WildAnimal extends Animal, Wild {
    public abstract void howl();
}

class Tiger implements WildAnimal {
    public void move() {
        System.out.println("성큼 성큼");
    }

    public void attack() {
        System.out.println("퍽!");
    }
    
    public void howl() {
        System.out.println("크앙!");
    }
}
```

또한, 인터페이스는 클래스와는 다르게 위의 `WildAnimal`과 같이 다중 상속이 가능하다.

### **인터페이스의 default 메소드**
인터페이스의 default 메소드에서의 `default`는 접근 제어자의 default (package-private)와는 다른 의미이다.  
부모 클래스에서 메소드를 구현해서 자식 클래스들에 해당 메소드의 기본 구현을 제공하는 것처럼, 인터페이스의 default 메소드는 구현 클래스에 기본 구현을 제공할 때 사용한다.

```java
interface Greeting {
    default void sayHello() {  // 인터페이스에서는 default 키워드를 명시적으로 사용 가능
        System.out.println("Hello from default method!");
    }
}

class Person implements Greeting {
    // abstract 메소드와는 달리, default 메소드는 구현하지 않아도 됨
}

public class Main {
    public static void main(String[] args) {
        Person person = new Person();
        person.sayHello(); // Hello from default method!
    }
}
```

### **인터페이스 사용 이유**

- 구현 클래스들의 메소드를 통일된 방식으로 사용 가능 (**다형성**)
- 추상 클래스보다 더 유연하기 때문에 코드 결합도를 낮출 수 있음
- java는 클래스의 다중 상속을 지원하지 않지만, 인터페이스를 통해 다중 상속 효과를 낼 수 있음
- 유지보수, 확장성 증가

---
## **추상 클래스 vs 인터페이스**

### **공통점**

1. 객체 생성 불가능  
2. 인터페이스를 구현하거나 추상 클래스를 상속한 클래스는 추상 메소드를 반드시 구현해야 함
3. 상위 클래스/인터페이스에 정의된 메소드를 재정의(Overriding)할 때, 접근 제어자를 더 개방적인 접근 제어자로 변경 가능

### **차이점**

|                          | 추상 클래스 |                인터페이스                 |
| :----------------------: | :---------: | :---------------------------------------: |
|       사용 키워드        |  abstract   |                 interface                 |
|  사용 가능 접근 제어자   |  제한 없음  |              public, default              |
|      사용 가능 변수      |  제한 없음  |        public static final (상수)         |
|     사용 가능 메소드     |  제한 없음  | public abstract, static, default, private |
|     상속/구현 키워드     |   extends   |                implements                 |
| 다중 상속/구현 가능 여부 |   불가능    |                   가능                    |
