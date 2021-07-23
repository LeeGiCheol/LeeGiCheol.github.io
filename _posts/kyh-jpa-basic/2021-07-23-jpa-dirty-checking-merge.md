---
title : "JPA 변경감지와 병합"
categories : 
    - jpa
tag :
    - jpa
toc : true
---

[김영한 : 실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1) 강의를 공부한 내용입니다.  

---

#### **JPA Update**   
JPA는 알다시피 Update 쿼리를 날릴 수 있는 메서드가 따로 없다.  
보통 persist를 호출하면 내부적으로 Insert와 Update를 구분해서 쿼리를 실행하는데,  
어떤 동작으로 실행되는지 알아보려한다.  

---

#### **Dirty Checking**  
JPA를 공부했다면 Dirty Checking이란 용어를 들어봤을 것이다.  
JPA는 트랜잭션안에서 커밋된 시점에 flush를 한다.  
바로 이 시점에 DB의 데이터와 엔티티의 데이터의 변경을 감지해서  
Update를 실행해준다.  

주의사항으로는 Dirty Checking의 대상은 영속 상태이며,  
준영속 상태, 비영속 상태는 Dirty Checking의 대상이 아니다.  

```java
@Transactional
@GetMapping("/")
public ResponseEntity<Users> updateUser() {
    Users user = new Users();
    user.setName("LEE");

    Users saveUser = usersRepository.save(user);
    saveUser.setName("GICHEOL");

    return ResponseEntity.ok(saveUser);
}
```

```shell
curl http://localhost:8080/        
{"id":1,"name":"GICHEOL"}
```

간단한 예시이다.  
LEE라는 이름으로 User를 저장하고, GICHEOL이란 이름으로 변경했다.  
명시적으로 Update를 호출하진 않았지만,  
해당 URL 요청을 하게되면 유저의 이름이 변경된 것을 알 수 있다.  

---

#### **준영속 상태의 엔티티 Update**  
준영속 상태는 영속성 컨텍스트가 더 이상 관리하지 않는 엔티티이다.  
즉 이전에는 영속 상태였던 엔티티이다.  

```java
@PostMapping("/books/edit")
public String updateBook(@ModelAttribute BookForm bookForm) {
    Book book = modelMapper.map(bookForm, Book.class);

    bookService.saveBook(book);
    return "redirect:/books";
}
```

일반적으로 웹사이트에서 Update를 할때는  
기존의 데이터를 먼저 화면에 보여주고,  
수정을 할 수 있도록 화면을 구성한다.  

위 예제도 마찬가지라고 가정한다.  

이미 DB에 들어가있는 데이터를 먼저 화면에 보여준 후,  
데이터를 수정한 후 BookForm 객체에 담아서 PostMapping으로 받아온다.  

참고로 여기서 사용된 modelMapper는  
VO to DTO , DTO to VO 등 객체 변환을 할때 유용하게 사용할 수 있다.  

```java
ModelMapper를 사용하지 않는 경우

@PostMapping("/books/edit")
public String updateBook(@ModelAttribute BookForm bookForm) {
    Book book = new Book();
    book.setId(bookForm.getId);
    book.setName(bookForm.getName);
    ...

    bookService.saveBook(book);
    return "redirect:/books";
}
```

다시 본론으로 돌아와서  
Book은 영속성 컨텍스트에 관리 대상이 아니다.  
그러나, 관리 대상인 Book의 Id 즉 식별자를 가지고 있다.  
이러한 경우 Book은 준영속 상태이다.  

---

##### **준영속 상태 엔티티 Dirty Checking**   
위의 예제의 Book은 준영속 상태이다.  
Dirty Checking의 대상이 되려면 영속상태로 변경해주면 된다.  

```java
@Controller
@RequiredArgsConstructor
public class BookController {
        
    @PostMapping("/books/edit")
    public String updateBook(@ModelAttribute BookForm bookForm) {
        Book book = bookService.updateBook(bookForm);
        return "redirect:/books";
    }

}
```

```java

@Service
@RequiredArgsConstructor
public class BookService {

    private final BookRepository bookRepository;
    private final ModelMapper modelMapper;

    @Transactional
    public void updateBook(BookForm bookForm) {
        Book findBook = bookRepository.findOne(bookForm.getId());
        modelMapper.map(bookForm, findBook);
    }

}
```

영속상태로 변경하기 위해 BookForm의 식별자인 Id로  
조회를 해서 Book 객체를 만들었다.  
Book은 이로써 영속상태이며, Dirty Checking의 대상이 된 것이다.  
modelMapper를 통해 변경이 된 객체를 set 해주면  
@Transactional 이 붙어있으므로, 트랜잭션이 커밋되어 flush가 발생할 때  
Dirty Checking에 의해 Update가 실행된다.  

---

##### **Merge**
Merge는 준영속 상태의 엔티티를 영속상태로 병합을 해준다.  

지금 공부중인 것은 그냥 JPA이지만  
잠깐 Deep Dive를 하게되었다.   

이후에 공부할 Spring Data JPA는 JpaRepository라는 인터페이스를 제공한다.  

이 인터페이스의 구현체는 SimpleJpaRepository인데,  
여기 save 메서드가 구현되어 있어  
우리는 별 다른 구현없이도 쉽게  
Spring Data JPA가 제공하는 save 메서드를 사용할 수 있다.  

```java
@Transactional
@Override
public <S extends T> S save(S entity) {

    Assert.notNull(entity, "Entity must not be null.");

    if (entityInformation.isNew(entity)) {
        em.persist(entity);
        return entity;
    } else {
        return em.merge(entity);
    }
}
```

save 시점에 isNew 메서드를 호출해서  
true면 persist를 false면 merge를 한다.  

isNew 메서드는 많은 interface와 구현체가 있고,  
코드도 너무 복잡해서 정확한 해석은 하지 못했다..  

<br>  

EntityInformation.class
```java
boolean isNew(T var1);
```

JpaMetamodelEntityInformation.class
```java
public boolean isNew(T entity) {
    if (this.versionAttribute.isPresent() && !(Boolean)this.versionAttribute.map(Attribute::getJavaType).map(Class::isPrimitive).orElse(false)) {
        BeanWrapper wrapper = new DirectFieldAccessFallbackBeanWrapper(entity);
        return (Boolean)this.versionAttribute.map((it) -> {
            return wrapper.getPropertyValue(it.getName()) == null;
        }).orElse(true);
    } else {
        return super.isNew(entity);
    }
}
```

AbstractEntityInformation.class
```java
public boolean isNew(T entity) {
    ID id = this.getId(entity);
    Class<ID> idType = this.getIdType();
    if (!idType.isPrimitive()) {
        return id == null;
    } else if (id instanceof Number) {
        return ((Number)id).longValue() == 0L;
    } else {
        throw new IllegalArgumentException(String.format("Unsupported primitive id type %s!", idType));
    }
}
```

그나마 이해한 것은 아래와 같다.  

isNew 메서드는 1차캐시 또는 DB에 조회 후 해당 엔티티의 식별자가  
Reference Type의 경우 Null,  
Primitive Type의 경우 0이라면 true를 리턴,  
아니라면 false를 리턴한다.  

isNew 메서드의 결과가 true라면 persist, false라면 merge를 한다.

---

##### **준영속 상태 엔티티 Merge**  
다시 JPA로 돌아와서 Merge에 대해 알아보겠다.  

```java
@Controller
@RequiredArgsConstructor
public class BookController {

    private final BookService bookService;

    @PostMapping("/books/edit")
    public String updateBook(@ModelAttribute BookForm bookForm) {
        Book book = bookService.createBook(bookForm);
        bookService.saveBook(book);
        return "redirect:/books";
    }

}
```

```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class BookService {

    private final BookRepository bookRepository;
    private final ModelMapper modelMapper;


    @Transactional
    public Long saveBook(Book book) {
        bookRepository.save(book);
        return book.getId();
    }

    public Book createBook(BookForm bookForm) {
        return modelMapper.map(bookForm, Book.class);
    }

}
```

```java
@Repository
@RequiredArgsConstructor
public class BookRepository {

    private final EntityManager em;


    public void save(Book book) {
        if (book.getId() == null) {
            em.persist(book);
        } else {
            em.merge(book);
        }
    }

}
```

Controller - Service - Repository 계층의 구성이다.  
먼저 Repository의 save 메서드를 보면  
Book 엔티티를 파라미터로 받고,  
book.getId()의 null 체크를 한다.  
null이면 persist, 아니라면 merge를 한다.  
(SimpleJpaRepository와 비슷한 구성)  

이때 merge가 된다는건,  
Book 엔티티가 가진 식별자를 통해 조회되는  
DB의 데이터에 새로 변경된 Book 데이터를 병합시킨다는 의미이다.  

즉 Insert가 아닌 Update이다.  

여기서 주의해야할 부분이 있다.  

EntityManager.persist는 반환타입이 void이다.  
EntityManager.merge는 반환타입이 파라미터로 받은 엔티티의 타입이다.  

영속성 컨텍스트의 관리 대상과 연관이 있는데, 간단한 테스트로 확인해보자.    

EntityManager.contains는  
해당 엔티티가 영속성 컨텍스트에서 관리되고 있는지 확인할 수 있다.  

```java

@Repository
@RequiredArgsConstructor
public class BookRepository {

    private final EntityManager em;

    public void save(Book book) {
        if (book.getId() == null) {
            System.out.println("before : " + em.contains(book));
            em.persist(book);
            System.out.println("after : " + em.contains(book));
        } else {
            System.out.println("before book : " + em.contains(book));
            book mergeBook = em.merge(book);
            System.out.println("after book : " + em.contains(book));
            System.out.println("after mergeBook : " + em.contains(mergeBook));
        }
    }
}

// persist
before : false
after : true

// merge
before book : false
after book : false
after mergeBook : true
```

즉 persist를 호출하면 파라미터에 들어간 값이 영속 상태가 되고,  
merge를 호출하면 파라미터에 들어간 값은 그대로 준영속 상태이며,  
반환된 엔티티가 영속 상태가 된다는 것을 알 수 있다.  

다시 Spring Data JPA를 확인해보자.  

SimpleJpaRepository.class
```java
@Transactional
@Override
public <S extends T> S save(S entity) {

    Assert.notNull(entity, "Entity must not be null.");

    if (entityInformation.isNew(entity)) {
        em.persist(entity);
        return entity;
    } else {
        return em.merge(entity);
    }
}
```

persist를 한 경우 파라미터로 받은 entity를 반환,  
merge를 한 경우 merge를 하고 받은 entity를 반환하는 것을 알 수 있다.  

---

###### **Merge 주의사항**  
Dirty Checking을 하면 변경된 데이터만 선택해서 변경할 수 있다.  
그러나 Merge는 모든 속성이 변경된다.  
변경될 엔티티의 한 필드에 값이 없다면  
해당 필드는 DB에 null로 업데이트 된다.   

보통은 Update문을 사용할 때  
전체 데이터를 사용하는 것보단, 필요한 데이터만 변경할 일이 훨씬 많다..  

이런 위험성이 있어 Merge를 사용하는 것보단,  
DirtyChecking을 사용하는 것이 더 안전하다.  

---