---
title : "[스프링 빈과 의존관계]"

excerpt : "컴포넌트 스캔과 자동 의존관계 설정, 자바 코드로 직접 스프링 빈 등록하기"

categories:
- Spring

tags: 
- [Blog, Spring, Github, Git, Web, example, backend]
  

toc: true
toc_sticky: true
pin: true
---

## 컴포넌트 스캔과 자동 의존관계 설정

- 스프링 빈을 등록하는 두 가지 방법으로

  1. 컴포넌트 스캔과 자동 의존관계 설정

  2. 자바 코드로 직접 스프링 빈에 등록하는 방법  


   
<br/>


서비스, 도메인 리포지토리를 만들었고 이제는 화면을 붙이고 싶다. 

그렇다면 Controller와 View를 정의해야 한다.

 
먼저, 멤버 컨트롤러를 만들어야 하는데 멤버 컨트롤러가 멤버 서비스를 통해서 회원 가입을 하고 데이터를 조회할 수 있어야 한다. 이 구조에서 의존관계가 형성되는데 멤버 컨트롤러가 멤버 서비스에 의존한다고 표현한다.

 
가령 멤버 컨트롤러에서 멤버 서비스를 다음과 같이 생성할 수 도 있다. 하지만 이와 같이 멤버 서비스를 생성하는 것은 문제가 있는데,  다른 여러 컨트롤러들이 멤버 서비스를 사용할 수 있고 각각의 컨트롤러마다 서로 다른 인스턴스를 사용하게 될 것이다.

 하지만 멤버 서비스는 여러 인스턴스를 사용할 필요 없이 하나의 인스턴스만 생성해서 공용으로 사용해도 되는 인스턴스이다.

 ```java
 @Controller
public class MemberController {
    private final MemberService memberService = new MemberService(...);
}
 ```

 따라서 직접 객체를 생성하는 것이 아닌, 외부로부터 객체를 받게 되면 자연스럽게 의존도가 낮아진다.

 ```java
 public MemberController(MemberService memberService) {
    this.memberService = memberService;
}
 ```

 이처럼 외부로부터 객체를 주입받는 것을 DI(Dependency Injection)이라고 하는데, 스프링은 스프링 컨테이너라는 곳에 객체를 빈으로 보관했다고 필요할 때 객체를 주입하는 DI 기능을 제공한다.

 

다음과 같이 @Autowired를 붙여주면 스프링 컨테이너에 등록된 memberservice를 주입해 준다.

그렇다면 memberService는 어떻게 컨테이너의 빈에 등록하는가? 바로 @Service 어노테이션을 붙여주면 된다.

```java
@Autowired
public MemberController(MemberService memberService) {
    this.memberService = memberService;
}
```

@Component 어노테이션이 붙으면 스프링은 해당 클래스의 객체를 자동으로 빈으로 등록한다.

마찬가지로 @Controller, @Service, @Repository 어노테이션도 내부적으로 @Component 어노테이션을 포함하기 때문에 스프링 빈으로 자동 등록된다.

![image](https://github.com/taeyoung0/git-cli/assets/115425415/36cdd8f7-f154-4085-ae29-60f17d2856dd)


@SpringBootApplication을 실행하면 속해있는 패키지 내의 @Controller, @Service, @Repository를 통해 각각이 컨테이너에 등록되고, @Autowired를 통해 의존관계를 가지는 모습이다. Controller를 통해 외부 요청을 받고, Service에서 비즈니스 로직을 처리, Repository에서 데이터를 저장하는 것이 위에서 언급한 정형화된 패턴이라고 할 수 있다.

 

참고로 스프링은 스프링 컨테이너에 스프링 빈 등록 시 중복되지 않게 유일하게 하나의 객체만 싱글톤으로 등록한다. 예를 들어 따로 설정을 하는 경우가 아니라면 다른 컨트롤러(또는 서비스)에서 사용하는 리포지토리는 같은 리포지토리 인스턴스를 사용한다.


## 자바 코드로 직접 스프링 빈 등록하기

Configuration 클래스에서 직접 스프링 빈을 등록해 보자.

기존에 작성했던 회원 서비스와 회원 리포지토리의 @Service, @Repository 어노테이션은 제거하고, SpringConfig 클래스를 생성하고 @Configuration 어노테이션을 붙여주면 기본 설정은 끝난다.

 

객체를 생성하고 반환하는 메서드에 @Bean 어노테이션만 붙여주면 스프링 컨테이너에 빈의 형태로 객체가 등록된다.

```java
@Configuration
public class SpringConfig {

    @Bean
    public MemberService memberService(){
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository(){
        return new MemoryMemberRepository();
    }

}
```

- 클래스명 앞에 @Configuration을 등록하면 스프링이 설정 파일임을 인식, 컨테이너에 스프링 빈을 등록할 준비

- @Bean을 사용하여 다음의 메서드를 호출해 스프링 빈이 등록.
  
-  memberService메서드가 호출되면 MemberService 객체가 생성되고, 이때 @Bean을 통해 컨테이너에 등록된 레포지토리가 매개변수로 회원 서비스 객체와 연결(@Autowired와 비슷), 회원 서비스 객체가 반환(스프링 빈으로 등록).  
  
<br/>

컨트롤러는 컴포넌트 스캔과 동일하게 작동한다. 즉, @Controller로 스프링 컨테이너에 등록, @Autowired로 스프링 빈에 등록된 객체와 연결.

 
위와 같이 설정 파일(자바 코드)을 직접 등록하여 스프링 빈을 등록할 수 있다. 

 
DI에는 크게 필드 주입, setter 주입, 생성자 주입의 3가지 방법이 있다.

- 필드 주입은 생성자 없이 @Autowired로 DI를 주입하는 방법 

- setter 주입은 setter에 @Autowired로 연결하는 방법

- 생성자 주입은 위에서 살펴본 것처럼 new를 사용하여 생성자를 호출할 때 DI를 호출하는 방법

<br/>
<br/>

일반적인 프로젝트에서는 조립(생성자 호출) 시점에 생성자를 한 번만 호출하여 컨테이너에 스프링 빈을 등록하고, 이후에는 의존관계가 동적으로 변경되는 경우가 거의 없기 때문에 생성자 주입이 주로 권장된다.

 

또한 실무에서는 주로 정형화된 컨트롤러, 서비스, 리포지토리 같은 코드는 컴포넌트 스캔을 사용하고, 정형화되지 않거나 상황에 따라 구현 클래스를 변경해야 할 경우 설정(@Configuration)을 통해 스프링 빈으로 등록한다.

 강의에서는 임시로 사용하고 있는 MemoryMemberRepository를 나중에 실제 DB로 변경할 예정이기 때문에 컴포넌트 스캔 방식 대신 자바 코드로 스프링 빈을 설정했다. (실제로 SpringConfig 파일의 new MemoryMemberRepository();를 new DBMemberRepository();로 변경하여 간단히 리포지토리를 변경 가능.)

 

주의해야 할 점은 @Autowired를 사용한 DI는 MemberController, MemberService와 같이 스프링이 관리하는 객체에서만 동작한다. 즉, 스프링 빈으로 등록되어 스프링 컨테이너에 올라가야만 @Autowired가 동작한다. 예를 들어 @Configuration으로 설정 파일은 생성되어 있지만 @Bean으로 서비스, 리포지토리가 등록되지 않은 경우이거나, 혹은 main() 등에서 직접 new를 사용하여 객체를 생성하더라도 이는 스프링 컨테이너에 올라간 객체가 아니므로(스프링이 관리하는 객체가 아니기 때문에) @Autowired는 동작하지 않는다.