---
title : "Java Builder Pattern"
categories : 
    - java
tag :
    - design-pattern
toc : true
---

---

#### **객체의 필드 초기화**
우리가 보통 객체를 선언하고 필드를 초기화 하기 위해선 크게 두 가지 방법이 있다.  
첫 번째로는 setter 메서드를 사용하는 것 이고,  
두 번째로는 생성자를 사용하는 것 이다.  

---

##### **setter**  

```java
public class User {

    private String username;

    private String password;

    private int age;

    private String address;

    private String hobby;

    
    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public void setHobby(String hobby) {
        this.hobby = hobby;
    }
    
}
```

위와 같은 POJO 클래스에 setter 메서드를 설정 후  
필요에 따라 필드에 값을 줄 수 있다.  

```java
public static void main(String[] args) {
    User user = new User();
    user.setUsername("LEEGICHEOL");
    user.setPassword("PASSWORD");
    user.setAge(26);
}
```

이와 같은 방식은 코드가 너무 길어지며, 중복이 많아진다는 단점이 있다.  

---

##### **Constructor**

다음은 생성자를 통한 초기화 방법이다.  

```java
public class User {

    private String username;

    private String password;

    private int age;

    private String address;

    private String hobby;

    public User() {
    }

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    public User(String username, String password, int age, String address, String hobby) {
        this.username = username;
        this.password = password;
        this.age = age;
        this.address = address;
        this.hobby = hobby;
    }
    
}
```

```java
public static void main(String[] args) {
    User softUser = new User("LEEGICHEOL", "PASSWORD");
    User optionalCheckedUser = new User("LEE", "PASSWORD", 26, "Running");
}
```

이와 같은 생성자 방식은 setter에 비해 훨씬 깔끔하지만,  
오히려 코드의 가로 길이가 늘어나는 단점과,  
필드에 따라 생성자가 따로 있어야 하며,  
순서가 정해져 있으므로 실수할 가능성이 있다.

---

#### Builder Pattern
사실 상 단순한 경우라면 setter나 생성자를 통해 충분히 가능하지만  
빌더 패턴을 사용한다면 실제 객체는 숨기고  
작업이 완료 되었을 때에만 객체를 호출 할 수 있어 불안정한 객체를 사용하지 않을 수 있다.    
필요에 따라 각 필드별로 프로세스/검증 등의 행위를 추가하기 좋다.  

```java
public class User {

    private String username;

    private String password;

    private int age;

    private String address;

    private String hobby;

    public User() {
    }

    public User(String username, String password, int age, String address, String hobby) {
        this.username = username;
        this.password = password;
        this.age = age;
        this.address = address;
        this.hobby = hobby;
    }
    
}
```

```java
public interface UserBuilder {

    UserBuilder username(String username);

    UserBuilder password(String password);

    UserBuilder age(int age);

    UserBuilder address(String address);

    UserBuilder hobby(String hobby);

    User build();
    
}
```

```java
public class RoleUserBuilder implements UserBuilder {

    private String username;

    private String password;

    private int age;

    private String address;

    private String hobby;


    @Override
    public UserBuilder username(String username) {
        this.username = username;
        return this;
    }

    @Override
    public UserBuilder password(String password) {
        this.password = password;
        return this;
    }

    @Override
    public UserBuilder age(int age) {
        this.age = age;
        return this;
    }

    @Override
    public UserBuilder address(String address) {
        this.address = address;
        return this;
    }

    @Override
    public UserBuilder hobby(String hobby) {
        this.hobby = hobby;
        return this;
    }

    @Override
    public User build() {
        return new User(username, password, age, address, hobby);
    }
    
}
```

위와 같이 UserBuilder 인터페이스에 각 필드를 저장할 수 있도록 메서드를 만들고,  
RoleUserBuilder처럼 구현을 한다.  
이때 자기자신을 return 함으로써 체이닝 구조를 가질 수 있다.  

```java
public static void main(String[] args) {
    UserBuilder userBuilder = new RoleUserBuilder();
    
    User user = userBuilder.username("LEEGICHEOL")
                            .password("PASSWORD")
                            .age(26)
                            .address("ANYANG")
                            .hobby("RUNNING")
                            .build();
}
```

build 라는 메서드를 호출하지 않으면 User 객체는 사용하지 못한다.   

---

##### UserBuilder에 필드 제거하기
```java

public class RoleUserBuilder implements UserBuilder {

    private User user;

    public UserBuilder newInstance() {
        this.user = new User();
        return this;
    }

    @Override
    public UserBuilder username(String username) {
        this.user.setUsername(username);
        return this;
    }

    @Override
    public UserBuilder password(String password) {
        this.user.setPassword(password);
        return this;
    }

    @Override
    public UserBuilder age(int age) {
        this.user.setAge(age);
        return this;
    }

    @Override
    public UserBuilder address(String address) {
        this.user.setAddress(address);
        return this;
    }

    @Override
    public UserBuilder hobby(String hobby) {
        this.user.setHobby(hobby);
        return this;
    }

    @Override
    public User build() {
        return new User(
                this.user.getUsername(),
                this.user.getPassword(),
                this.user.getAge(),
                this.user.getAddress(),
                this.user.getHobby());
    }

}
```

```java
public static void main(String[] args) {
    UserBuilder userBuilder = new RoleUserBuilder().newInstance();

    User user = userBuilder.username("LEEGICHEOL")
            .password("PASSWORD")
            .age(26)
            .address("ANYANG")
            .hobby("RUNNING")
            .build();
}
```

newInstance 메서드를 통해 객체 생성을 먼저 한 후  
setter를 통해 필드의 값을 초기화한다.  
이런 방법으로 필드의 중복을 제거할 수있다.  

한번 더 나아가서 Director를 사용할 수도 있다.  

---

##### **Director**

```java
public class UserDirector {

    private final UserBuilder userBuilder;

    public UserDirector(UserBuilder userBuilder) {
        this.userBuilder = userBuilder;
    }

    public User getUser() {
        return userBuilder
                .username("LEEGICHEOL")
                .password("PASSWORD")
                .age(26)
                .address("ANYANG")
                .hobby("RUNNING")
                .build();
    }

}
```

```java
public static void main(String[] args) {
    UserDirector userDirector = new UserDirector(new RoleUserBuilder().newInstance());
    User user = userDirector.getUser();
}

```

Builder를 한번 더 숨기고, Director를 통해 User를 생성하는 방식이다.  
필요에 따라 newInstance 메서드는 UserDirector 생성자에서 호출할 수도 있다.  

---

##### **장점**
빌더 패턴의 장점으로는  
setter나 생성자의 비해 코드의 양이 줄어들고, 직관적인 코드를 작성할 수 있다.  
또한 RoleUserBuilder 대신 RoleAdminBuilder를 만들어서 바꿔 끼워준다면  
유저의 권한별로 유저 계정의 차이를 두게 끔 구현 할 수 있다.  

---

##### **단점**
이러한 장점에 비해 구현을 위해 User 객체를 만들기 전에  
Director 혹은 Builder 객체를 생성해야 한다는 것과  

구조가 오히려 복잡해진다는 단점이 있다.  

---

##### **극복**
사실 Lombok 라이브러리가 적용되지않은 실무 프로젝트는 드문 편이다.  
빌더패턴이 이러한 구조를 통해 필드 초기화를 할 수 있다는 것을 알아두고,  
라이브러리를 쓰지 못하는 환경이 아니라면, Lombok을 활용하는 것도 나쁘지 않다.  

```java
@Data
@Builder
public class User {

    private String username;

    private String password;

    private int age;

    private String address;

    private String hobby;

}
```

```java
public static void main(String[] args) {
    User user = User.builder()
                    .username("LEEGICHEOL")
                    .password("PASSWORD")
                    .age(26)
                    .address("ANYANG")
                    .hobby("RUNNING")
                    .build();
} 
```

---