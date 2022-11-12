## 아이템 42. 익명 클래스보다는 람다를 사용하라

### 시작하기 전
자바 8 에서는 함수형 인터페이스, 람다, 메서드 참조라는 개념이 추가되면서 함수 객체를 더 쉽게 만들 수 있게 됐다. 아이템 42에서는 변경 가능성을 줄여 안정성을 높이고, 가독성을 높이는 방식에 대해 알아본다.

### 함수 객체, 익명 클래스, 람다를 활용한 방식 비교
예전에는 함수 타입을 표현할 때 메서드를 하나만 담은 인터페이스(드물게는 추상클래스)를 사용했다.

#### 함수 객체란
1. 인터페이스의 인스턴스 함수를 의미한다.
2. 특정 함수나 동작을 나타나는 데 사용한다.

#### 익명 클래스란
1. 함수 기능을 사용하는 곳에서 클래스를 구현하는 방식을 의미한다.

#### 람다란
1. 자바 8에서 추상 메서드 하나만 존재하는 인터페이스가 특별한 대우를 받아 탄생했다.
2. 인스턴스를 생성하지 않고 기능 만을 구현해 사용할 수 있다.

> 익명 클래스를 사용하면 코드가 길어지는 문제점이 있었다.

> 람다를 활용하면 동작과 관련되지 않은 동작들을 구현하지 않기 때문에 코드가 명확해진다.

### 자바에서 함수형 프로그래밍

#### 함수 객체를 활용한 방법

```java
@Test  
void stringCompareTestUsingFunctionObject() {  
    StringComparator comparator = new StringComparator();  
    Assertions.assertEquals(comparator.compare("abc", "bcd"), -1);  
}  
  
private static class StringComparator implements Comparator<String> {  
    @Override  
    public int compare(String a, String b) {  
        return a.compareTo(b);  
    }  
}
```

#### 익명함수를 활용한 방법

```java
@Test  
void stringCompareTestUsingAnonymousClass() {  
    Comparator<String> anonymousComparator = new Comparator<>() {  
        @Override  
        public int compare(String a, String b) {  
            return a.compareTo(b);  
        }  
    };  
    Assertions.assertEquals(anonymousComparator.compare("abc", "bcd"), -1);  
}
```

#### 람다를 활용한 방법

```java
@Test  
void stringCompareTestUsingLambda() {  
    Comparator<String> anonymousComparator = (a, b) -> a.compareTo(b);  
    Assertions.assertEquals(anonymousComparator.compare("abc", "bcd"), -1);  
}
```

#### 메서드 참조를 활용한 방법

```java
@Test  
void stringCompareTestUsingLambdaOmittedParameters() {  
    Comparator<String> anonymousComparator = String::compareTo;  
    Assertions.assertEquals(anonymousComparator.compare("abc", "bcd"), -1);  
}
```

### 람다를 사용할 때 주의할 점

#### 주의할 점 1 : 람다와 타입 추론

람다를 사용할 때부터 반환값을 언급하지 않는다. 이게 가능한 이유는 컴파일러가 문맥을 살펴 타입을 추론해주기 때문이다. 상황에 따라 컴파일러가 타입을 결정하지 못할 경우 프로그래머가 직접 명시해야 한다.

> 결론 : 반환값을 언급해야 하는 경우는 컴파일러가 추론하지 못할 때, 명확성을 위해 타입을 명시해야하는 경우이다.

> 컴파일러가 타입을 추론하는 데 필요한 타입 정보 대부분을 제네릭에서 얻는다.

#### 주의할 점 2 : 람다는 일시적이다.

람다는 이름이 없고 문서화할 수 없다. 따라서 코드 자체로 동작이 명확하게 설명되지 않거나 코드 줄 수가 많아지면 사용하지 않는다.

> 같은 기능을 구현한 코드를 재사용하지 않는다면 람다를 만들어 가독성을 높이는 것이 좋지만, 재사용이 가능하거나 코드가 복잡해 문서화해야 할 경우 람다를 사용하지 않는다.

#### 주의할 점 3 : 람다로는 구현할 수 없는 경우가 존재한다.
람다로는 구현할 수 없는 곳에서는 익명 클래스를 사용하는 등 이전 방식을 사용해야 한다.

1. 추상 클래스의 인스턴스 생성
2. 추상 메서드가 여러 개 인터페이스의 인스턴스를 생성할 때
3. 자신을 참조하는 곳

> 람다는 자신을 참조할 수 없다. 람다에서 this 키워드는 바깥 인스턴스를 참조하는 반면 익명 클래스에서 this는 익명 클래스 인스턴스를 참조한다.

#### 주의 사항 4 : 람다는 구현한 벤더마다 직렬화 방식이 다르다.
람다 구현은 익명 클래스처럼 직렬화 형태가 다를 가능성이 있기 때문에 직렬화를 해서는 안된다. 직렬화해야 한다면 private 정적 중첩 클래스의 인스턴스를 사용하자.

### 마지막으로
- 자바 8로 버전 업이 되면서 작은 함수 객체를 구현하는데 람다가 도입 되었다.
- 익명 클래스는 타입의 인스턴스를 만들 때만 사용하라
- 람다는 작은 함수 객체를 쉽게 표현할 수 있어 실용성이 뛰어나다.


### 번외 : 제네릭과 람다에 대한 타입 추론

자바 8에서는 변수에 대한 타입 추론이 없기 때문에 자바에서 타입 추론이라 하면 제네릭과 람다에 대한 타입 추론을 의미한다.

```ad-note
컴파일러가 타입을 추론하는 데 필요한 타입 정보 대부분을 제네릭에서 얻는다. 우리가 이 정보를 제공하지 않으면 컴파일러는 람다의 타입을 추론할 수 없게 되어 일일이 명시해줘야 한다.

Effective Java Item 42 - 255p
```

제네릭은 컴파일 타임에 소거가 되기 때문에 런타임에서는 타입을 추론할 수 없기 때문에 람다를 지원하기 위해 타입 추론이 개선되었다.

#### 메서드 호출 시 인자 타입 추론
자바8 이전에 제네릭 타입의 인자를 넘겨주는 경우 타입 추론이 안됐지만, 자바 8부터는 가능하다.

```java
static void processNames(List<String> names) {  
    for (String name : names) {  
        System.out.println("Hello " + name);  
    }  
}  
  
@Test  
void test1() {  
    List<String> names = Collections.emptyList();  
    processNames(Collections.emptyList());  
}
```

> 자바 7 버전으로 실행하면 컴파일 에러가 발생한다.

#### 연쇄 메서드 호출 시
하지만, 특정한 메서드로 인해 타입이 제거된 후 새로운 메서드를 호출하는 경우 문제가 발생했다.

```java
List<String> list = List.emptyList().add(":("); // error
```

어쩔 수 없이 명시적으로 타입을 알려줄 필요가 있다.

```java
List<String> list = List.<String>emptyList().add(":("); // OK
```

컴파일러가 미리 체크할 수 있도록 `@FunctionalInterface` 어노테이션으로 표시해줄 수 있다.

```java
@FunctionalInterface  
public interface Comparator<T> {  
    int compare(T a, T b);  
}
```

> 함수형 인터페이스가 컴파일러에게 타입을 추론해주기 때문에 람다는 인자의 타입을 추론할 수 있다.

### 참고 자료
- [자바 8 - 함수형 인터페이스 이해하기](https://codechacha.com/ko/java8-functional-interface/)
- [Java Lambda (2) 타입 추론과 함수형 인터페이스](https://futurecreator.github.io/2018/07/20/java-lambda-type-inference-functional-interface/)


