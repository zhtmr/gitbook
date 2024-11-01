# 상속관계 매핑

## 상속관계 매핑

> 객체의 상속 관계를 관계형 데이터베이스에서 표현하는 방법에 대해 알아본다.

<figure><img src="https://github.com/user-attachments/assets/0a70aad6-b4c1-42f0-9d07-a6d3b2885770" alt=""><figcaption></figcaption></figure>

### 1. 조인 전략

<figure><img src="https://github.com/user-attachments/assets/1ab8420c-2d57-43f3-8fcd-e54a7f962042" alt=""><figcaption></figcaption></figure>

* 공통 데이터(NAME, PRICE) 등은 ITEM 테이블에 넣고, 나머지 데이터는 각각의 엔티티에 해당하는 테이블 컬럼에 넣는다.
* ITEM 테이블에서 DTYPE 컬럼을 통해 해당 레코드가 어떤 엔티티의 정보인지 구분한다.
* ALBUM / MOVIE / BOOK 등과 같은 하위 테이블은 ITEM 의 PK 를 FK 로 가진다. (조인 컬럼으로 쓰기위해)

#### Item 클래스

```java
@Entity
@Getter @Setter
@Inheritance(strategy = InheritanceType.JOINED)   // 여기서 전략을 설정한다.
@DiscriminatorColumn                              // DTYPE 컬럼이 생성된다.
public abstract class Item {

  @Id @GeneratedValue
  private Long id;

  private String name;
  private int price;
}
```

#### Movie 클래스

```java
@Entity
@Getter @Setter
public class Movie extends Item {        // Item 클래스를 상속받는다.

  private String director;
  private String actor;
}
```

실행하면 jpa 가 각각의 엔티티를 테이블로 만든다.

<figure><img src="https://github.com/user-attachments/assets/1fae55b8-cf46-422d-84e9-16e9fd984203" alt="" width="375"><figcaption></figcaption></figure>

그 후 각각의 테이블에 Item 테이블과 조인을 위한 외래키를 설정한다.

<figure><img src="https://github.com/user-attachments/assets/73f99d46-fba1-42f8-931c-75ff94faa426" alt="" width="375"><figcaption></figcaption></figure>

#### INSERT & SELECT 테스트

<figure><img src="https://github.com/user-attachments/assets/7595d441-87db-43d8-927b-5839bddaaff4" alt="" width="375"><figcaption></figcaption></figure>

아래와 같이 각각의 테이블에 insert 된다. (insert 두번 된다.)

select 시에는 join 해서 가져온다.

<figure><img src="https://github.com/user-attachments/assets/a579913f-5a7d-4aec-b20c-2d1f4e50aba7" alt="" width="375"><figcaption></figcaption></figure>

실제 테이블에도 fk 가 잘 매핑되어 있다.

<figure><img src="https://github.com/user-attachments/assets/a6251f60-4769-45b9-9201-e9c481ea58f3" alt=""><figcaption></figcaption></figure>

Item 클래스에 `@DiscriminatorColumn` 컬럼을 추가하면 `DTYPE` 컬럼이 생성되는데 해당 컬럼에 엔티티 이름이 들어간다. 부모 테이블에서 해당 레코드가 어떤 자식테이블의 데이터인지 알 수 있다. 조인 전략에서는 `@DiscriminatorColumn` 가 없어도 자식 테이블을 쿼리해보면 누구의 데이터인지 알 수 있지만 단일 테이블 전략을 사용할 경우엔 필수로 해당 어노테이션을 추가해야한다.

<figure><img src="https://github.com/user-attachments/assets/b074c15d-7069-4456-b646-d9fe004bc589" alt=""><figcaption></figcaption></figure>

### 2. 단일 테이블 전략

<figure><img src="https://github.com/user-attachments/assets/02b7553e-39dc-43df-83b5-afbf0279e41d" alt=""><figcaption></figcaption></figure>

* 모든 정보를 한 테이블에 몰아 넣고 DTYPE 컬럼을 통해 구분한다.
* NAME, PRICE 와 같은 공통 데이터 컬럼을 제외하고 나머지 컬럼은 모두 null 허용으로 해야한다. (엔티티마다 안쓰이는 컬럼이 있기 때문)

#### Item 클래스

```java
@Entity
@Getter @Setter
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)   // 여기서 전략을 설정한다.
public abstract class Item {

  @Id @GeneratedValue
  private Long id;

  private String name;
  private int price;
}
```

Item 클래스에서 전략을 `SINGLE_TABLE` 로 설정할 경우 `MOVIE` / `ALBUM` / `BOOK` 과 같은 자식테이블을 만들지 않고, ITEM 테이블에 모든 컬럼을 추가한다. 컬럼들이 null 허용되어 있어서 안쓰는 컬럼엔 null 로 들어간다.

단일 테이블 전략을 사용하게되면 `@DiscriminatorColumn` 을 Item 클래스에 추가하지 않아도 자동으로 `DTYPE` 컬럼이 생성된다.

<figure><img src="https://github.com/user-attachments/assets/849d003d-6512-459c-b3a4-b496646aa83d" alt="" width="375"><figcaption></figcaption></figure>

<figure><img src="https://github.com/user-attachments/assets/a67f5436-28fb-4e5f-8985-3a238a57cb1d" alt=""><figcaption></figcaption></figure>

#### INSERT & SELECT 테스트

<figure><img src="https://github.com/user-attachments/assets/bbd17b3f-e4cf-4594-a9ee-4aa3ea615271" alt="" width="375"><figcaption></figcaption></figure>

단일 테이블은 insert 가 한번만 호출되고 select 쿼리도 심플하다. jpa 는 기본전략으로 단일 테이블 전략을 사용한다.

### 3. 구현 클래스마다 테이블 전략 (실제로 잘 안쓰임)

<figure><img src="https://github.com/user-attachments/assets/8d85d741-d1ad-43cf-b63a-7490293e35af" alt=""><figcaption></figcaption></figure>

* 아무런 참조 관계 없이 각각의 테이블에 저장하는 방식
* 관계를 맺기 힘들어서 거의 쓰이지 않음
* 부모 타입으로 조회시 모든 자식테이블을 조회하고 union all 한다.
* Item 테이블 안 만들어진다.

#### Item 클래스

```java
@Entity
@Getter @Setter
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)   // 여기서 전략을 설정한다.
@DiscriminatorColumn                              // DTYPE 컬럼이 생성된다.
public abstract class Item {

  @Id @GeneratedValue
  private Long id;

  private String name;
  private int price;
}
```

`TABLE_PER_CLASS` 전략은 상속 관계 없이 그냥 각각의 테이블을 만들고 해당 데이터는 그 테이블에 넣는다.

<figure><img src="https://github.com/user-attachments/assets/63f069a3-2afc-4dea-b8d4-d9b67c8f3861" alt="" width="375"><figcaption></figcaption></figure>

#### INSERT

<figure><img src="https://github.com/user-attachments/assets/43d57357-195d-419b-be72-89089b7bea64" alt="" width="375"><figcaption></figcaption></figure>



#### SELECT

아래와 같이 부모 타입으로 조회할 경우 모든 자식테이블의 결과를 union all 로 묶어서 반환한다.

```java
Item movie1 = em.find(Item.class, movie.getId());
```

```sql
Hibernate: 
    select
        i1_0.id,
        i1_0.clazz_,
        i1_0.name,
        i1_0.price,
        i1_0.artist,
        i1_0.author,
        i1_0.isbn,
        i1_0.actor,
        i1_0.director 
    from
        (select
            price,
            id,
            artist,
            name,
            null as author,
            null as isbn,
            null as actor,
            null as director,
            1 as clazz_ 
        from
            Album 
        union
        all select
            price,
            id,
            null as artist,
            name,
            author,
            isbn,
            null as actor,
            null as director,
            2 as clazz_ 
        from
            Book 
        union
        all select
            price,
            id,
            null as artist,
            name,
            null as author,
            null as isbn,
            actor,
            director,
            3 as clazz_ 
        from
            Movie
    ) i1_0 
where
    i1_0.id=?
```

### 장점과 단점

* 조인 전략
  * 장점 : 테이블 정규화
  * 단점 : 조회 시 조인을 많이 사용, 테이블 저장 시 insert 두번 호출
* 단일 테이블 전략
  * 장점 : 조인이 필요 없어 조회 성능이 좋다, 조회 쿼리가 단순함
  * 단점 : 자식 엔티티의 컬럼은 모두 null 허용 필수, 테이블이 커질 수 있어 조회 성능이 오히려 느려질 수 있다.
* 구현 클래스마다 테이블 전략
  * 단점 : 장점이 없다.
