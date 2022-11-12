## 타입 안전 이종 컨테이너를 고려하라

### 제네릭을 사용한 컨테이너의 한계
제네릭은 보통 컬렉션과 단일원소 컨테이너에 사용되고, 매개변수화 대상은 컨테이너 자신이다. 
따라서 하나의 컨테이너에서 매개변수화할 수 있는 타입의 수가 제한된다.

### 타입 안전 이종 컨테이너(Type safe heterogeneous container pattern)
컨테이너가 아닌 키를 매개변수화 해 컨테이너에 값을 관리할 때, 매개변수화한 키를 함께 제공하면 된다. 이러한 설계 방법을 타입 안전 이종 컨테이너 패턴이라 한다.

> 즉, 다른 타입들을 모든 컨테이너를 안전하게 관리하는 방법에 대해서 설명한다.

### 타입 안전 이종 컨테이너 패턴 활용 예

1. 컴파일 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴(타입 토큰)을 사용한다.
2. 값을 저장할 때 타입을 검사하고, 값을 가져올 때에도 타입을 검사하기 떄문에 타입 안전하다.

```java
public class Favorites {  
    private final Map<Class<?>, Object> favorites = new HashMap<>();  
  
    public <T> void putFavorite(Class<T> type, T instance) {  
        favorites.put(Objects.requireNonNull(type), type.cast(instance));  
    }  
  
    public <T> T getFavorite(Class<T> type) {  
        return type.cast(favorites.get(type));  
    }  
}
```

#### 참고 1 : 와일드 카드를 활용한 맵의 키 타입
비한정적 와일드카드 타입이라 맵 안에 아무것도 넣을 수 없다 생각할 수 있지만, 와일드카드 타입이 중첩되어 있어서 값을 넣을 수 있다. 
즉, 맵이 아니라 키가 와일드 카드인 것이다.

```java
private final Map<Class<?>, Object> favorites
```

> 와일드카드 타입의 특성인 다양한 타입을 지원하는 기능을 활용한 방법이다.

#### 참고 2 : Object 타입인 맵의 값 타입
이 맵은 키와 값 사이 타입 관계를 보증하지 않는다는 것을 알 수 있다. 하지만 set과 get 메서드에서 타입 안정성을 검사하고 있기 때문에 안정적라는 것을 유추할 수 있다.

> 타입 이종 컨테이너 패턴을 사용할 때, 타입 안정성을 고려해 구현해야 한다.

### 타입 이종 컨테이너를 구현할 때 주의점

#### 주의점 1 : 로 타입으로 값을 넘기면 타입 안정성이 쉽게 깨진다.
로 타입으로 값을 넘긴다면 타입 안정성이 깨지기 쉬운데, 컨테이너가 타입 불변식을 어기는 일이 없도록 보장하기 위해서는 값이 저장될 때, 인스턴스의 타입을 미리 확인한다.

```java
HashSet<Integer> instance = new HashSet<>();  
// SuppressedWaring !!  
((HashSet) instance).add("문자열이다.");
```

CheckedCollection 클래스 내부에는 typeCheck 메서드를 이용해 값을 저장하는 순간 타입을 검사하게 된다.

```java
static class CheckedCollection<E> implements Collection<E>, Serializable {
	final Collection<E> c;  
	final Class<E> type;

	public boolean add(E e) { 
		return c.add(typeCheck(e)); 
	}
	
	@SuppressWarnings("unchecked")  
	E typeCheck(Object o) {  
	    if (o != null && !type.isInstance(o))  
	        throw new ClassCastException(badElementMsg(o));  
	    return (E) o;  
	}

	private String badElementMsg(Object o) {  
	    return "Attempt to insert " + o.getClass() +  
	        " element into collection with element type " + type;  
	}
	...
}
```


#### 주의점 2 : 실체화 불가 타입에는 사용할 수 없다.
제네릭을 사용하는 클래스를 넣기 위해서 매개변수화하지 않은 클래스 리터럴을 이용해야 한다. 매개변수가 다른 컨테이너로 값을 가져온다면 오류가 발생하게 된다.

```java
Favorites favorites = new Favorites();  
HashSet<String> instance = new HashSet<>();  
favorites.putFavorite(HashSet.class, instance);  
  
// SuppressedWaring !!  
HashSet<Integer> set = favorites.getFavorite(HashSet.class);
```

두 번째 제약에 대한 완벽한 우회로는 없다. 방법 중 하나로는 슈퍼 타입 토큰을 이용해 시도할 수 있다.

> 스프링 프레임워크에서는 ParameterizedTypeReference라는 클래스로 미리 구현했고, 아래와 같이 사용할 수 있다.

```java
List<String> instance = Arrays.asList("개", "고양이", "앵무");
favorites.putFavorite(new TypeRef<List<String>>(){}, instance);  

List<String> pets = favorites.getFavorite(new TypeRef<List<String>>(){});
```

슈퍼 타입 토큰도 완벽하지 않은데, 그 이유는 모든 상황에서 타입 안정성을 완벽하게 지키지 못하기 때문이다.  아래와 같이 경고를 제거하게 되면, 다른 타입으로 값을 가져오는 경우에도 경고가 발생하지 않게 된다.

```java
@SuppressWarnings("unchecked")  
public <T> T getFavorite(TypeRef<T> type) {  
    return (T) favorites.get(type);  
}
```

> 다른 타입으로 가져오게되면 컴파일 에러가 발생하지 않고 런타임에서 문제가 발생한다.

### 한정적 와일드 카드를 사용해 더욱 안정성있게 구현하자
특정한 타입만 토큰 키로 구현하고 싶은 경우 한정적 타입 매개변수나 한정적 와일드카드를 사용해 표현 가능한 타입을 제한할 수 있다.

### 애너테이션 API에서 타입 토큰 사용
애너테이션 API는 한정적 타입 토큰을 이용해 타입을 확인하고 있다.

#### AnnotatedElement#getAnnotation
아래 메서드는 각 요소마다 달린 애너테이션을 런타임에 읽는 기능이다. 이 메서드가 읽는 요소들은 클레스와 메서드 필드와 같은 요소이다. 

```java
<T extends Annotation> T getAnnotation(Class<T> annotationClass);
```

> 눈치챌 수 있듯이 annotationClass가 타입 토큰이 되고, 토큰으로 명시한 타입의 애너테이션이 대상 요소에 달리면 애너테이션을 반환하고, 그렇지 않다면 null을 반환한다.

객체를 `Class<? extends Annotation>`이라 선언할 수 있지만, 컴파일 경고가 뜨기 때문에 해당 메서드에는 안전하고 동적으로 수행해주는 인스턴스를 사용하고 있다. `annotationType.asSubclass(Annotation.class)` 방식 처럼 런타임에 읽어 오류나 경고 없이 컴파일이 된다.

```java
@SuppressWarnings("unchecked")  
public <U> Class<? extends U> asSubclass(Class<U> clazz) {  
    if (clazz.isAssignableFrom(this))  
        return (Class<? extends U>) this;  
    else        throw new ClassCastException(this.toString());  
}
```

### 마지막 정리
- 가바에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정되어 있지만, 키를 타입 매개 변수로 바꾸면 이런 제약이 없는 타입 안전 이종 컨테이너를 구현할 수 있다.
- 타입 이종 안전 컨테이너는 키를 class 리터럴로 사용한다.
- 컨테이너를 담는 경우 ParameterizedTypeReference를 이용한 슈퍼 타입 토큰을 이용할 수 있지만, 이마저도 완벽하게 안전하지 않다.
- asSubclass 메서드를 이용해 타입 불완전 경고를 보지 않고 구현할 수 있다.
