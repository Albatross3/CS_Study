> All persistent classes must have a default constructor (which can be non-public) so that Hibernate can instantiate them using Constructor.newInstance(). It is recommended that you have a default constructor with at least package visibility for runtime proxy generation in Hibernate.
>

→ persistent class(엔티티 클래스인듯) 는 기본 생성자를 반드시 가져야 하며 package visibility 를 가져야 한다 (Hibernate 에서 런타임 프록시 생성을 위해서)
## 엔티티 클래스가 기본 생성자가 있어야 하는 이유

스프링에서 @RequestBody 를 통해 객체(DTO) 에 매핑할 때 DTO 는 기본 생성자를 가져야 했었다. 이는 스프링의 @RequestBody 바인딩 방식이 기본 생성자를 통해 객체를 생성한 후 `Java Reflection` 을 이용해 필드 값을 넣어주는 방식이기 때문이다.

- `Reflection` 을 이용해 DTO 에 매핑하는 방식

  Refelection 은 클래스 이름만 알면 필드, 메서드, 생성자 등 클래스의 모든 정보에 접근이 가능하다. 하지만 접근할 수 없는 정보가 `생성자의 매개변수 정보` 이다. 따라서 모든 필드를 받는 생성자가 있더라도 Reflection 은 해당 생성자를 호출할 수 없게 된다. 그래서 Reflection 은  기본 생성자로 객체를 생성하고 필드 값을 강제로 매핑해주는 방식을 사용한다.


위와 같이 JPA 역시 DB 에서 데이터를 가져온 뒤 객체를 생성할 때 `Reflection` 을 사용한다. 실제로 빈 생성자와 완전한 생성자에 각각 다른 로그를 찍어놓고 테스트해보면 기본 생성자에 만들어 놓은 로그만 찍히는 것을 볼 수 있다.

- `Reflection` 을 사용하는 이유

  이는 우리가 엔티티로 어떤 타입을 생성할 지 JPA는 알 수 없기 때문입니다. Reflection을 사용하지 않고 객체를 생성하려면 미리 객체의 타입을 알고 있어야 합니다. **하지만 프레임워크나 라이브러리는 사용자가 정의할 구체 클래스 정보를 알 수가 없습니다.** 때문에 어떤 타입으로 엔티티를 만들더라도 해당 엔티티를 생성하기 위해 Reflection을 사용하여 엔티티 인스턴스를 만들어 주는 것입니다.


## 기본 생성자가 private 이면 안되는 이유

Reflection 으로 객체를 생성한다면 @RequestBody 의 용도로 사용할 클래스들의 기본 생성자의 접근 제한자를 private 으로 제한하는 것처럼 엔티티의 기본 생성자 역시 private 로 하면 안되는가?

엔티티 객체의 생성 자체만 생각하면 옳은 얘기이다. 하지만 `Lazy Loading` 개념을 생각한다면 틀린 얘기이다. `Lazy Loading` 은 연관된 엔티티에 프록시 객체(가짜 객체) 를 넣어둔 뒤 해당 객체의 정보가 필요한 타이밍에 연관된 엔티티를 조회해오는 방식이다. 이 때 프록시 객체를 사용하기 위해서는 엔티티 생성자가 private 이면 안된다.

엔티티의 프록시는 원본 엔티티를 상속해서 만들게 된다. 지연 로딩으로 인해 프록시 객체를 넣어줘야 하면 원본 엔티티를 상속한 프록시 엔티티를 만들고, 실제 필요한 타이밍에 엔티티를 조회해 온 뒤 프록시 엔티티가 원본 엔티티를 참조하도록 하여 사용한다.