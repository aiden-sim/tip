## Lombok 사용시 주의점

롬복을 사용하면 DTO, Domain 등을 만들 때 Getter, Setter, 생성자 등 반복적인 부분을 애노테이션으로 줄여줍니다.

IDE에서 Code generation을 사용하면 Getter, Setter, 생성자를 자동으로 만들어 주긴 하지만 롬복을 사용할 때보다 코드가 길어집니다.

롬복의 단점으로는 IDE에 별도 플러그인 설치가 필요하고, 간단하게 애노테이션 형태로 사용할 수 있기 때문에 객체 생성 시, 실수할 소지가 있다고 생각됩니다.

</br>

#### 1) \@AllArgsConstructor, \@RequiredArgsConstructor

AllArgsConstructor는 전체 필드의 생성자를 RequiredArgsConstructor는 final이나 \@NonNull인 필드 대상으로 생성자를 만들어 줍니다.

문제는 필드의 순서가 변경되면, 생성자의 파라미터 순서도 자동 변경돼서, 동일한 필드의 타입인 경우 컴파일 시점에서 에러를 발견하기 힘듭니다.

그렇기 때문에 생성자는 명시적으로 생성해 주는 것이 좋습니다.

</br>

#### 변경 전
``` java
@AllArgsConstructor
@ToString
public class User {
  private String driverId;
  private String mobileNo;
  private String driverName;
}

User user = new User("simjunbo", "01088253765", "심준보");
결과 : User(driverId=simjunbo, mobileNo=01088253765, driverName=심준보)
```

</br>

#### 변경 후
``` java
@AllArgsConstructor
@ToString
public class User {
  private String mobileNo; // mobileNo와 driverId 순서 변경
  private String driverId;
  private String driverName;
}

User user = new User("simjunbo", "01088253765", "심준보");
결과 : User(mobileNo=simjunbo, driverId=01088253765, driverName=심준보) // mobileNo와 driverId의 값이 변경 됨
```

</br>

#### 2) \@Data

``` java
/**
* @see Getter
* @see Setter
* @see RequiredArgsConstructor
* @see ToString
* @see EqualsAndHashCode
* @see lombok.Value
*/
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
public @interface Data {
  String staticConstructor() default "";
}
``` 

Data 애노테이션을 사용하면 내부적으로 \@Getter, \@Setter, \@RequiredArgsConstructor, \@ToString, \@EqualsAndHashCode 등 많은 기능들을
제공합니다.

**\@RequiredArgsConstructor** 같은 경우 위에서 설명한 것처럼 필드 순서가
변경되면 이슈가 될 수 있습니다.

</br>

또한 **\@EqualsAndHashCode**은 equals랑 hashCode 메서드를 자동으로 생성해 주는데
mutable한 객체에서 사용하면 부작용이 생길 수 있습니다.

\@Data는 필드에 대한 Setter를 자동 생성해 주기 때문에 mutable한 객체가
생성됩니다.

``` java
User user = new User("simjunbo", "01088253765", "심준보");
User user2 = new User("simjunbo", "01088253765", "심준보");

user.equals(user2);
결과 : true

user2.setMobileNo("01011111111"); // 특정 필드 변경
user.equals(user2);
결과 : false
``` 

</br>

#### 3) \@Builder

\@Builer를 클래스에 사용하게 되면  \@AllArgsConstructor를 default type(동일패키지 접근)으로 생성한 효과가 발생됩니다.

동일 패키지 내에서 전체 필드에 대한 생성자를 통해 접근 가능하게 되고 위에서 설명한 것처럼 필드 순서가 변경되면 이슈가 될 수 있습니다.

#### 변경 전

``` java
@Builder
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class User {
  private String driverId;
  private String mobileNo;
  private String driverName;
}

// 컴파일 시점에 다음과 같이 변경됨
public class User {
  private String mobileNo;
  private String driverId;
  private String driverName;
  
  User(String mobileNo, String driverId, String driverName) {
    this.mobileNo = mobileNo;
    this.driverId = driverId;
    this.driverName = driverName;
}
... (생략)
}
``` 

</br>

생성자를 직접 만들어서 \@Builder를 사용하면 필드 순서 변경에 따른 이슈도 사라지고, 외부에 노출이 필요한 필드를 직접 제어할 수 있게 됩니다.

#### 변경 후
``` java
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class User {
  private String driverId;
  private String mobileNo;
  private String driverName;
  
  @Builder
  private User(String mobileNo, String driverName) {
    this.driverId = UUID.randomUUID().toString();
    this.mobileNo = mobileNo;
    this.driverName = driverName;
  }
}
```

driverId는 자동으로 생성되기 때문에 외부에 노출시키지 않았습니다.

![img/lombok/289d9fedaaad4b2e1481f63c8c0a8f1d](img/lombok/50335beb9a8b90ac2fac44e4a0484ba9.tmp)

</br>

#### 4) \@NoArgsConstructor

JPA를 사용할 때, 프록시 생성을 위해 기본 생성자가 필요하다고 합니다. 
기본 생성자가 없으면 아래와 같이 No default constructor for entity 메시지가 출력됩니다.

``` java
[2019-04-17 19:01:17] [ERROR]c.t.t.a.c.r.HttpExceptionHandlerResolver.doExceptionProcess[48] HttpExceptionHandlerResolver :
org.springframework.orm.jpa.JpaSystemException: No default constructor for entity: : com.sjb.AccidentReportRemainOrder;
nested exception is org.hibernate.InstantiationException: No default constructor for entity: : com.sjb.AccidentReportRemainOrder
at org.springframework.orm.jpa.vendor.HibernateJpaDialect.convertHibernateAccessException(HibernateJpaDialect.java:333)
at org.springframework.orm.jpa.vendor.HibernateJpaDialect.translateExceptionIfPossible(HibernateJpaDialect.java:244)
. . .
```

</br>

다음과 같이 \@NoArgsConstrucotr 접근 권한을 PROTECTED로 해서 외부에서 생성자를 통해 불필요한 객체를 생성할 수 없도록 처리할 수 있습니다.

``` java
@Entity
@Table(name = "auth")
@Getter
@ToString
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Auth {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  @Column(name = "auth_seqno", nullable = false)
  private Long authSeqno;
  
  @Column(name = "driver_id", nullable = false, length = 100)
  private String driverId;
... (생략)
}
```

</br>

#### 5) \@Setter
Setter는 객체를 mutable 하게 만들기 때문에 가급적 지양하려고 하지만,
DTO 같은 경우 자동으로 파싱 해주는 라이브러리에 따라서 Setter가 있어야 데이터가 주입되는 경우가 있습니다.
그래서 여러 프로젝트에서 사용되는 DTO인 경우 Setter를 유지하려고 합니다.

해당 방법이 꼭 정답이라고 할 수는 없지만, 롬복 사용 시 고민해 볼 필요는 있을 것 같습니다.

</br>

참고)

<https://www.popit.kr/%EC%8B%A4%EB%AC%B4%EC%97%90%EC%84%9C-lombok-%EC%82%AC%EC%9A%A9%EB%B2%95/>

<https://github.com/cheese10yun/spring-jpa-best-practices/blob/master/doc/step-06.md>

<http://kwonnam.pe.kr/wiki/java/lombok/pitfall>

 
