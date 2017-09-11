***
# 5. 스프링 웹 애플리케이션 만들기
* 요청을 스프링 컨트롤러에 매핑하기
* 파라미터 폼을 투명하게 바인딩 하기
* 제출 폼 검증하기


* MVC 패턴을 기반으로, 스프링 MVC는 스프링 플임워크 자체와 긴밀한 연결없이 유연한 웹기만 애플리케이션을 만드는 데 도움을 줌( loosely coupled & 유연한)
* 다양한 웹 요청, 파라미터, 폼form 입력 등을 처리하는 컨트롤러를 만들기 위한 애너테이션 사용법을 집중적으로 다룬다.


***
## 5.1 스프링 MVC 시작하기
* 스프링은 요청(request)을 디스패처 서블릿(dispatcher servlet), 핸들러 매핑(handler mapping), 컨트롤러, 뷰 리졸버 (view resolver) 등으로 이동시킴

### 5.1.1 스프링 MVC를 이용한 요청 추적
![01](img/01.jpg)
그림 5.1 원하는 결과를 얻기 위한 여러 단계에 대한 요청 처리 정보

1. url, form 등 브라우저를 통해 요청 정보를 보내면 DispatcherServlet로 전달
2. DispatcherServlet은 컴포넌트를 선택하기위해 핸들러 매핑에게 위임
	* DispatcherServlet : Front Controller Pattern 임. Clinet요청을 Dispatcher한 곳에서 처리 하여 Controller를 실행하는 구조
3. 핸들러 매핑 : 핸들러 매핑들 > 컨트롤러 선택 > 선택된 컨트롤러에 3에 요청을 보냄
	* 핸들러 매핑은 HTTP 요청정보를 이용해서 이를 처리할 핸들러 오브젝트, 즉 컨트롤러를 찾아 주는 기능
	* Spring은 기본적으로 5개의 핸들러 매핑을 제공(디폴트 2개)
	* BeanNameUrlHandlerMapping : 빈의 이름에 들어 있는 URL을 HTTP 요청의 URL 과 비교해서 일치하는 빈을 찾아 줌
	* ControllerBeanNameHandlerMapping : 빈의 아이디나 빈 이름을 이용해 매핑해주는 핸들러 매핑 전략
	* http://springsource.tistory.com/3 [Rednics Blog]
4. 컨트롤러 : 요청정보를 처리 > 모델을 패키징 & 논리적 뷰의 이름 > DispatcherSeπlet 에게 Return
	* 모델 : 결과는 사용자의 브라우저에 표시되기 위한 형태의 정보로 반환
5. DispatcherServlet은 뷰 리졸버 에게 논리적으로 주어진 뷰의 이름과 실제로 구현된 뷰를 매핑해 줄 것을 요청
6. JSP로 모벨 데이터를 전달해 주는 뷰를 구현
7. 모델 데이터를 사용하여 결과를 렌더링하고 응답(response) 객체에 의해 클라이언트로 전달된다.





***
### 5.1.2 스프링 MVC 설정하기

** DISPATCHERSERVLET 설정하기 **

* DispatcherServlet은 스프링 MVC의 핵심
* 서블릿 3 스펙이나 스프링 3,1 이전엔 WAR 파일 안에 전달되는 web.xml이 유일
* java 설정은 서블릿 3.0 이상을 지원하는 서버에 배포될때만 동작함.
* 예제에서는 서블릿 컨테이너의 DispatcherServlet을 설정하는 데 web.xml을 사용하는 대신 Java를 사용



```java
public class SpitterWebInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
  @Override
  protected String[] getServletMappings() { //DispatcherServlet을/에매핑
    return new String[] { "/" };
  }

  @Override
  protected Class<?>[] getRootConfigClasses() {
    return new Class<?>[] { RootConfig.class };
  }

  @Override
  protected Class<?>[] getServletConfigClasses() { //설정클래스를영시
    return new Class<?>[] { WebConfig.class };
  }
}
```


***


** 두 애플리케이션 컨텍스트에 대한 이야기 **

* DispatcherServlet이 시작되면서 스프링 애플리케이션 컨텍스트를 생성하고 이를 클래스나 설정 파일로 선언된 빈으로 로딩하기 시작한다.
* DispatcherServlet : 컨트롤러, 뷰 리졸벼, 핸들러 매핑과 같은 웹 컴포넌트가 포함된 빈을 로딩할 것으로 예상
* ContextLoaderListener : 그 외의 다른 빈을 로딩할 것으로 예상(백 엔드를 구동하는 중간 계층 및 데이터 계층 구성 요소)
* 내부적으로 AbstractAnnotationConfigDispatcherServletlnitializer는 DispatcherServlet과 ContextLoaderListener를 생성한다.

** 이전 방식인 web.xml의 대안 **

```java
public class SpitterWebInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

  @Override
  protected Class<?>[] getRootConfigClasses() {
    return new Class<?>[] { RootConfig.class };  // 리턴된 @Configuration 클래스들은 ContextLoaderListener가 생성한 애플리케이션 컨텍스트를 설정하기 위해 사용된다.
  }

//DispatcherServlet이 애플리케이션 컨텍스트를 WebConfig 설정 클래스(Java 설정 사용)에서 정의된 빈으로 로딩하기를 요청
  @Override
  protected Class<?>[] getServletConfigClasses() {
    return new Class<?>[] { WebConfig.class }; //리턴된 @Configuration 클래스들은 DispatcherServlet의 애플리 케이션 컨텍스트에 대한 빈들을 정의한다
  }

  @Override
  protected String[] getServletMappings() {
    return new String[] { "/" };
  }
}
```


***


** 스프링 MVC 활성화하기 ** p167

* xml : < mvc:annotation-driven>요소가 애너테이션 드리븐(annotation-driven) 스프링 MVC를 활성화하는 데 사용 http://hamait.tistory.com/322
* 자바 기반 : 클래스를 작성할 때 @EnableWebMvc애너테이션을 붙여 주는 것이다.


* 뷰 리졸버가 설정되지 않은 경우 : 스프링은 디폴트로 빈의 ID와 View 인터페이스를 구현한 뷰의 이름을 매칭시켜 뷰를 결정하는 BeanNameViewResolver를 사용한다.
* 컴포넌트 검색 기능이 비활성화되어 있다. 결과적으로 스프링이 컨트롤러를 찾는 유일한 방법은 명시적으로 설정에 해당 컨트롤러들을 선언하는 것이다.
* 앞서 선언한 대로, DispatcherServlet은 애플리케이션의 디폴트 서블릿으로 매핑되어 이미지나 스타일 시트 같은 고정적인 리소스들에 대한 “모든” 요청을 처리한다(대부분의 경우에서 이런 동작들을 원하지 않는)

P 168 - 예제
```java
@Configuration
@EnableWebMvc //Spring MVC 활성화
@ComponentScan("spittr.web") //component-scanning 활성화
public class WebConfig extends WebMvcConfigurerAdapter {

  @Bean
  public ViewResolver viewResolver() {
    InternalResourceViewResolver resolver = new InternalResourceViewResolver(); //JSP 뷰 리출버 설정
    resolver.setPrefix("/WEB-INF/views/");
    resolver.setSuffix(".jsp");
    return resolver;
  }

  @Override
  public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) { //정적 콘텐츠 처리 실정
    configurer.enable();
  }
}
```

* @ComponentScan 애너테이션: spitter.web 패키지에서 컴포넌트를 스캔
	* 컨트롤러들에는 컴포넌트 스캐닝의 대상으로 만들어 주는 @Controller 애너테이션을 붙임
* ViewResolver 빈 : 뷰 리졸버는 특정한 접두사(prefiα)와 접미사(suffix)가 사용된 뷰 이름을 가지고 JSP 파일들을 찾아 주는 역할을 한다(예를 들면, home이라는 뷰 이름은 /WEB-INF/views/homeβp라는 JSP 파일로 결정

***

## 5.2 간단한 컨트롤러 작성하기

* @RequestMapping : 컨트롤러가 처리하게 될 요청의 종류를 정의
	* value 애트리뷰트는 이 메소드가 처리할 요청 패스를 명시하고 method 애트리뷰트로 처리할 수 있는 HTTP 메소드를 상세하게 기술

* return 하는 string : 이 String은 스프링 MVC에 의해 렌더링할 뷰의 이름으로 해석
	* DispatcherServlet은 뷰 리졸벼에게 이 논리적인 뷰의 이름으로 실제 뷰를 결정하도록 요청
	* InternalResource View Resolver 설정 방식으로 ”home”이라는 뷰 이름은 /WEB- INF/view/home.jsp 위 치에 있는 JSP 파일로 결정



```java
@Controller
public class HomeController {

  @RequestMapping(value="/", method = GET)  // "/"에 대한 GET요청을 처리
  public String home() {
    return "home"; //뷰의 이름은 "home"
  }
}
```


P172 실행하기



***

### 5.2.1 컨트롤러 테스팅


```java

@Test
	public void testHome() throws Exception {
		HomeController controller = new HomeController();
//		assertEquals("home", controller.home());

		MockMvc mockMvc = standaloneSetup(controller).build(); //Mo야Mvc 셋업
		mockMvc.perform(get("/")) //GET / 수행
				.andExpect(view().name("home"));  //home뷰를 보여중
	}
```


***

### 5.2.2 클래스 레벨 요청 처리 정의하기

```java
@Controller
@RequestMapping("/")  //@RequestMapping({"/", "/homepage"})
public class HomeController {

  @RequestMapping(method = GET)
  public String home() {
    return "home";
  }
}
```

***

### 5.2.3 뷰에 모델 데이터 전달하기
p176 실행

179

***

## 5.3 요청입력받기

* 스프링 MVC는 클라이언트가 컨트롤러의 핸들러 메소드에 데이터를 전달해 줄 몇 가지 방법을 제공한다. 그 방법들은 아래와 같다.
• 쿼리 파라미터
• 폼파라미터
• 패스 변수

***

### 5.3.1 쿼리 파라미터 입력받기

### 5.3.2 패스 파라미터를 통한 입력받기

***

## 5.4 폼처리하기


### 5.4.2 폼검증하기

### valid table 5.1


### valid 197p
만약 검증에서 오류가 발견되면 processRegistration()의 파라미터로 전달받는 Errors 객체를
사용하여 오류를 확인힌-다(Errrors 파라미터가 검증될 @Vail id 애너테이션이 붙어 있는 파라미터 바
로 다음에 있다는 것이 중요하대. processRegistration()은 먼저 오류 유무를 확인하기 위해 Erros.
hasErrors( )를 호출한다.










