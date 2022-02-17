# 스프링 MVC 1편
## 강의

[스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)

## 정리

<details>
<summary>웹 서버, 웹 애플리케이션 서버</summary>
<div markdown="1">       

**웹 서버(Web Server)**

- HTTP 기반으로 동작
- 정적 리소스 제공, 기타 부가기능
- 정적(파일) HTML, CSS, JS, 이미지, 영상
- 예) NGINX, APACHE

**웹 애플리케이션 서버(WAS - Web Application Server)**

- HTTP 기반으로 동작
- 웹 서버 기능 포함 + (정적 리소스 제공 가능)
- 프로그램 코드를 실행해서 애플리케이션 로직 수행
    - 동적 HTML, HTTP API(JSON)
    - 서블릿, JSP, 스프링 MVC
- 예) 톰캣(Tomcat), Jetty, Undertow

**웹 서버, 웹 애플리케이션 서버 차이**

- 웹 서버는 정적 리소스(파일), WAS는 애플리케이션 로직
- 사실은 둘의 용어도 경계도 모호함
    - 웹 서버도 프로그램을 실행하는 기능을 포함하기도 함
    - 웹 애플리케이션 서버도 웹 서버의 기능을 제공함
- 자바는 서블릿 컨테이너 기능을 제공하면 WAS
    - 서블릿 없이 자바코드를 실행하는 서버 프레임워크도 있음
- WAS는 애플리케이션 코드를 실행하는데 더 특화

**웹 시스템 구성 - WAS, DB**

- WAS, DB 만으로 시스템 구성 가능
- WAS는 정적 리소스, 애플리케이션 로직 모두 제공 가능
- WAS가 너무 많은 역할을 담당, 서버 과부하 우려
- 가장 비싼 애플리케이션 로직이 정적 리소스 때문에 수행이 어려울 수 있음
- WAS 장애시 오류 화면도 노출 불가능

**웹 시스템 구성 - WEB, WAS, DB**

- 정적 리소스는 웹 서버가 처리
- 웹 서버는 애플리케이션 로직 같은 동적인 처리가 필요하면 WAS에 요청을 위임
- WAS는 중요한 애플리케이션 로직 처리 전담
- 효율적인 리소스 관리
    - 정적 리소스가 많이 사용되면 Web 서버 증설
    - 애플리케이션 리소스가 많이 사용되면 WAS 증설
- 정적 리소스만 제공하는 웹 서버는 잘 죽지 않음
- 애플리케이션 로직이 동작하는 WAS 서버는 잘 죽음
- WAS, DB 장애시 Web 서버가 오류 화면 제공 가능

</div>
</details>

<details>
<summary>서블릿</summary>
<div markdown="1">       

**특징**

```java
@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet {

	@Override
	protected void service(HttpServletRequest request, HttpServletResponse response) {
		//애플리케이션 로직
	} 
}
```

- urlPatterns(**/hello**)의 URL이 호출되면 서블릿 코드가 실행
- HTTP 요청 정보를 편리하게 사용할 수 있는 HttpServletRequest
- HTTP 응답 정보를 편리하게 제공할 수 있는 HttpServletResponse
- 개발자는 HTTP 스펙을 매우 편리하게 사용

**HTTP 요청, 응답 흐름**

- HTTP 요청시
    - WAS는 Request, Response 객체를 새로 만들어서 서블릿 객체 호출
    - 개발자는 Request 객체에서 HTTP 요청 정보를 편리하게 꺼내서 사용
    - 개발자는 Response 객체에 HTTP 응답 정보를 편리하게 입력
    - WAS는 Response 객체에 담겨있는 내용으로 HTTP 응답 정보를 생성

**서블릿 컨테이너**

- 톰캣처럼 서블릿을 지원하는 WAS를 서블릿 컨테이너라고 함
- 서블릿 컨테이너는 서블릿 객체를 생성, 초기화, 호출, 종료하는 생명주기 관리
- 서블릿 객체는 **싱글톤으로 관리**
    - 고객의 요청이 올 때 마다 계속 객체를 생성하는 것은 비효율
    - 최초 로딩 시점에 서블릿 객체를 미리 만들어두고 재활용
    - 모든 고객 요청은 동일한 서블릿 객체 인스턴스에 접근
    - **공유 변수 사용 주의**
    - 서블릿 컨테이너 종료시 함께 종료
- JSP도 서블릿으로 변환 되어서 사용
- 동시 요청을 위한 멀티 쓰레드 처리 지원


</div>
</details>

<details>
<summary>동시 요청 - 멀티 쓰레드</summary>
<div markdown="1">       

**쓰레드**

- 애플리케이션 코드를 하나하나 순차적으로 실행하는 것은 쓰레드
- 자바 메인 메서드를 처음 실행하면 main이라는 이름의 쓰레드가 실행
- 쓰레드가 없다면 자바 애플리케이션 실행이 불가능
- 쓰레드는 한번에 하나의 코드 라인만 수행
- 동시 처리가 필요하면 쓰레드를 추가로 생성

**요청 마다 쓰레드 생성 - 장단점**

- 장점
    - 동시 요청을 처리할 수 있다.
    - 리소스(CPU, 메모리)가 허용될 때 까지 처리 가능
    - 하나의 쓰레드가 지연 되어도, 나머지 쓰레드는 정상 동작한다.
- 단점
    - 쓰레드 생성 비용은 매우 비싸다.
        - 고객의 요청이 올 때 마다 쓰레드를 생성하면, 응답 속도가 늦어진다.
    - 쓰레드는 컨텍스트 스위칭 비용이 발생한다.
    - 쓰레드 생성에 제한이 없다.
        - 고객 요청이 너무 많이 오면, CPU, 메모리 임계점을 넘어서 서버가 죽을 수 있다.

**쓰레드 풀 - 요청 마다 쓰레드 생성의 단점 보완**

- 특징
    - 필요한 쓰레드를 쓰레드 풀에 보관하고 관리한다.
    - 쓰레드 풀에 생성 가능한 쓰레드의 최대치를 관리한다. 톰캣은 최대 200개 기본 설정(변경 가능)
- 사용
    - 쓰레드가 필요하면, 이미 생성되어 있는 쓰레드를 쓰레드 풀에서 꺼내서 사용한다.
    - 사용을 종료하면 쓰레드 풀에 해당 쓰레드를 반납한다.
    - 최대 쓰레드가 모두 사용중이어서 쓰레드 풀에 쓰레드가 없으면?
        - 기다리는 요청은 거절하거나 특정 숫자만큼만 대기하도록 설정할 수 있다.
- 장점
    - 쓰레드가 미리 생성되어 있으므로, 쓰레드를 생성하고 종료하는 비용(CPU)이 절약되고, 응답 시간이 빠르다.
    - 생성 가능한 쓰레드의 최대치가 있으므로 너무 많은 요청이 들어와도 기존 요청은 안전하게 처리할 수 있다.

**쓰레드 풀 - 실무 팁**

- WAS의 주요 튜닝 포인트는 최대 쓰레드(max thread) 수이다.
- 이 값을 너무 낮게 설정하면?
    - 동시 요청이 많으면, 서버 리소스는 여유롭지만, 클라이언트는 금방 응답 지연
- 이 값을 너무 높게 설정하면?
    - 동시 요청이 많으면, CPU, 메모리 리소스 임계점 초과로 서버 다운
- 장애 발생시?
    - 클라우드면 일단 서버부터 늘리고, 이후에 튜닝
    - 클라우드가 아니면 열심히 튜닝

**쓰레드 풀의 적정 숫자**

- 애플리케이션 로직의 복잡도, CPU, 메모리, IO 리소스 상황에 따라 모두 다름
- 성능 테스트
    - 최대한 실제 서비스와 유사하게 성능 테스트 시도
    - 툴: 아파치 ab, 제이미터, nGrinder

**WAS의 멀티 쓰레드 지원 - 핵심**

- 멀티 쓰레드에 대한 부분은 WAS가 처리
- **개발자가 멀티 쓰레드 관련 코드를 신경쓰지 않아도 됨**
- 개발자는 마치 **싱글 쓰레드 프로그래밍을 하듯이 편리하게 소스 코드를 개발**
- 멀티 쓰레드 환경이므로 싱글톤 객체(서블릿, 스프링 빈)는 주의해서 사용


</div>
</details>

<details>
<summary>SSR, CSR</summary>
<div markdown="1">       

**서버사이드 렌더링, 클라이언트 사이드 렌더링**

- **SSR - 서버 사이드 렌더링**
    - HTML 최종 결과를 서버에서 만들어서 웹 브라우저에 전달
    - 주로 정적인 화면에 사용
    - 관련기술: JSP, 타임리프 → **백엔드 개발자**
- **CSR - 클라이언트 사이드 렌더링**
    - HTML 결과를 자바스크립트를 사용해 웹 브라우저에서 동적으로 생성해서 적용
    - 주로 동적인 화면에 사용, 웹 환경을 마치 앱 처럼 필요한 부분부분 변경할 수 있음
    - 예) 구글 지도, Gmail, 구글 캘린더
    - 관련기술: React, Vue.js → **웹 프론트엔드 개발자**
- 참고
    - React, Vue.js를 CSR + SSR 동시에 지원하는 웹 프레임워크도 있음
    - SSR을 사용하더라도, 자바스크립트를 사용해서 화면 일부를 동적으로 변경 가능

**백엔드 개발자 입장에서 UI 기술**

- **백엔드 - 서버 사이드 렌더링 기술**
    - JSP, 타임리프
    - 화면이 정적이고, 복잡하지 않을 때 사용
    - 백엔드 개발자는 서버 사이드 렌더링 기술 학습 **필수**
- **웹 프론트엔드 - 클라이언트 사이드 렌더링 기술**
    - React, Vue.js
    - 복잡하고 동적인 UI 사용
    - 웹 프론트엔드 개발자의 전문 분야
- **선택과 집중**
    - 백엔드 개발자의 웹 프론트엔드 기술 학습은 **옵션**
    - 백엔드 개발자는 서버, DB, 인프라 등등 수 많은 백엔드 기술을 공부해야 한다.
    - 웹 프론트엔드도 깊이있게 잘 하려면 숙련에 오랜 시간이 필요하다.

</div>
</details>

<details>
<summary>자바 백엔드 웹 기술 역사</summary>
<div markdown="1">       

**자바 웹 기술 역사**

- 서블릿 - 1997
    - HTML 생성이 어려움
- JSP - 1999
    - HTML 생성은 편리하지만, 비즈니스 로직까지 너무 많은 역할 담당
- 서블릿, JSP 조합 MVC 패턴 사용
    - 모델, 뷰 컨트롤러로 역할을 나누어 개발
- MVC 프레임워크 춘추 전국 시대 - 2000년 초 ~ 2010년 초
    - MVC 패턴 자동화, 복잡한 웹 기술을 편리하게 사용할 수 있는 다양한 기능 지원
    - 스트럿츠, 웹워크, 스프링 MVC(과거 버전)

**현재 사용 기술**

- **애노테이션 기반의 스프링 MVC 등장**
    - @Controller
    - MVC 프레임워크의 춘추 전국 시대 마무리
- **스프링 부트의 등장**
    - 스프링 부트는 서버를 내장
    - 과거에는 서버에 WAS를 직접 설치하고, 소스는 War 파일을 만들어서 설치한 WAS에 배포
    - 스프링 부트는 빌드 결과(Jar)에 WAS 서버 포함 → 빌드 배포 단순화

**최신 기술 - 스프링 웹 기술의 분화**

- Web Servlet - Spring MVC
- Web Reactive - Spring WebFlux

**최신 기술 - 스프링 웹 플럭스(Web Flux)**

- **특징**
    - 비동기 넌 블러킹 처리
    - 최소 쓰레드로 최대 성능 - 쓰레드 컨텍스트 스위칭 비용 효율화
    - 함수형 스타일로 개발 - 동시처리 코드 효율화
    - 서블릿 기술 사용X
- **그런데**
    - 웹 플럭스는 기술적 난이도 매우 높음
    - 아직은 RDB 지원 부족
    - 일반 MVC의 쓰레드 모델도 충분히 빠르다.
    - 실무에서 아직 많이 사용하지는 않음 (전체 1% 이하)

**자바 뷰 템플릿 역사 - HTML을 편리하게 생성하는 뷰 기능**

- JSP
    - 속도 느림, 기능 부족
- 프리마커(Freemarker), Velocity(벨로시티)
    - 속도 문제 해결, 다양한 기능
- 타임리프(Thymeleaf)
    - 내추럴 템플릿: HTML의 모양을 유지하면서 뷰 템플릿 적용 가능
    - 스프링 MVC와 강력한 기능 통합
    - **최선의 선택**, 단 성능은 프리마커, 벨로시티가 더 빠름


</div>
</details>

<details>
<summary>스프링 MVC 구조</summary>
<div markdown="1">       

- 스프링 MVC는 프론트 컨트롤러 패턴으로 구현되어 있다.
- 스프링 MVC의 프론트 컨트롤러가 바로 디스패처 서블릿(DispatcherServlet)이다.
- 이 디스패처 서블릿이 바로 스프링 MVC의 핵심이다.

**동작 순서**

1. **핸들러 조회**: 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다.
2. **핸들러 어댑터 조회**: 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
3. **핸들러 어댑터 실행**: 핸들러 어댑터를 실행한다.
4. **핸들러 실행**: 핸들러 어댑터가 실제 핸들러를 실행한다.
5. **ModelAndView 반환**: 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 **변환**해서 반환한다.
6. **viewResolver 호출**: 뷰 리졸버를 찾고 실행한다.
    - JSP의 경우: InternalResourceViewResolver 가 자동 등록되고, 사용된다.
7. **View 반환**: 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환한다.
    - JSP의 경우 InternalResourceView(JstlView) 를 반환하는데, 내부에 forward() 로직이 있다.
8. **뷰 렌더링**: 뷰를 통해서 뷰를 렌더링 한다.

### HTTP 요청 데이터

**주로 다음 3가지 방법을 사용한다.**

- **GET - 쿼리 파라미터**
    - /url**?username=hello&age=20**
    - 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달
    - 예) 검색, 필터, 페이징 등에서 많이 사용하는 방식
    - 쿼리 파라미터는 ‘?’를 시작으로 보낼 수 있다. 추가 파라미터는 ‘&’로 구분하면 된다.
- **POST - HTML Form**
    - content-type: application/x-www-form-urlencoded
    - 메시지 바디에 쿼리 파라미터 형식으로 전달 username=hello&age=20
    - 예) 회원 가입, 상품 주문, HTML Form 사용
- **HTTP message body**에 데이터를 직접 담아서 요청
    - HTTP API에서 주로 사용, JSON, XML, TEXT
    - 데이터 형식은 주로 JSON 사용
    - POST, PUT, PATCH

**매핑 정보**

- @RestController
    - @Controller는 return값이 String이면 뷰 이름으로 인식된다. 그래서 **뷰를 찾고 뷰가 렌더링** 된다.
    - @RestController는 return값으로 뷰를 찾는 것이 아니라, **HTTP 메시지 바디에 바로 입력**한다. 따라서 실행 결과로 ok 메시지를 받을 수 있다. @ResponseBody와 관련이 있다.
- @RequestMapping("/hello-basic")
    - /hello-basic URL 호출이 오면 이 메서드가 실행되도록 매핑한다.
    - 대부분의 속성을 배열[] 로 제공하므로 다중 설정이 가능하다. {"/hello-basic", "/hello-go"}

**HTTP 요청 - 기본, 헤더 조회**

- HttpServletRequest
- HttpServletResponse
- HttpMethod : HTTP 메서드를 조회한다. org.springframework.http.HttpMethod
- Locale : Locale 정보를 조회한다.
- @RequestHeader MultiValueMap<String, String> headerMap
    - 모든 HTTP 헤더를 MultiValueMap 형식으로 조회한다
- @RequestHeader("host") String host
    - 특정 HTTP 헤더를 조회한다.
    - 속성
        - 필수 값 여부: required
        - 기본 값 속성: defaultValue
- @CookieValue(value = "myCookie", required = false) String cookie
    - 특정 쿠키를 조회한다.
    - 속성
        - 필수 값 여부: required
        - 기본 값: defaultValue
- MultiValueMap
    - MAP과 유사한데, 하나의 키에 여러 값을 받을 수 있다.
    - HTTP header, HTTP 쿼리 파라미터와 같이 하나의 키에 여러 값을 받을 때 사용한다.
        - **keyA=value1&keyA=value2**

**HTTP 요청 파라미터 - 쿼리 파라미터, HTML Form**

- HttpServletRequest 의 request.getParameter() 를 사용하면 다음 두가지 요청 파라미터를 조회할 수 있다.
- GET 쿼리 파리미터 전송 방식이든, POST HTML Form 전송 방식이든 둘다 형식이 같으므로 구분없이
조회할 수 있다. 이것을 간단히 **요청 파라미터(request parameter) 조회**라 한다.

**HTTP 요청 파라미터 - @RequestParam**

- 스프링이 제공하는 @RequestParam 을 사용하면 요청 파라미터를 매우 편리하게 사용할 수 있다.
- @RequestParam.required
    - 파라미터 필수 여부
    - 기본값이 파라미터 필수( true )이다.
- **주의! - 파라미터 이름만 사용** /request-param?username=
    - 파라미터 이름만 있고 값이 없는 경우 빈문자로 통과
- **주의! - 기본형(primitive)에 null 입력** /request-param 요청
    - @RequestParam(required = false) int age
    - null 을 int 에 입력하는 것은 불가능(500 예외 발생)
    - 따라서 null 을 받을 수 있는 Integer 로 변경하거나, 또는 다음에 나오는 defaultValue 사용
- 파라미터에 값이 없는 경우 defaultValue 를 사용하면 기본 값을 적용할 수 있다. 이미 기본 값이 있기 때문에 required 는 의미가 없다.
- defaultValue 는 빈 문자의 경우에도 설정한 기본 값이 적용된다. /request-param?username=
- 파라미터를 Map, MultiValueMap으로 조회할 수 있다.
- @RequestParam Map
    - Map(key=value)
- @RequestParam MultiValueMap
    - MultiValueMap(key=[value1, value2, ...] ex) (key=userIds, value=[id1, id2])
- 파라미터의 값이 1개가 확실하다면 Map 을 사용해도 되지만, 그렇지 않다면 MultiValueMap 을 사용하자.

**HTTP 요청 파라미터 - @ModelAttribute**

- 롬복 @Data
    - @Getter , @Setter , @ToString , @EqualsAndHashCode , @RequiredArgsConstructor 를 자동으로 적용해준다.
- 스프링MVC는 @ModelAttribute 가 있으면 다음을 실행한다.
    - HelloData 객체를 생성한다.
    - 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티를 찾는다. 그리고 해당 프로퍼티의 setter를
    호출해서 파라미터의 값을 입력(바인딩) 한다.
    - 예) 파라미터 이름이 username 이면 setUsername() 메서드를 찾아서 호출하면서 값을 입력한다.
- @ModelAttribute 는 생략할 수 있다. 그런데 @RequestParam 도 생략할 수 있으니 혼란이 발생할 수 있다.
- 스프링은 해당 생략시 다음과 같은 규칙을 적용한다.
    - String , int , Integer 같은 단순 타입 = @RequestParam
    - 나머지 = @ModelAttribute (argument resolver 로 지정해둔 타입 외)

**HTTP 요청 메시지 - 단순 텍스트**

- HTTP 메시지 바디의 데이터를 InputStream 을 사용해서 직접 읽을 수 있다.
- InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회
- OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력
- **HttpEntity**: HTTP header, body 정보를 편리하게 조회
    - 메시지 바디 정보를 직접 조회
    - 요청 파라미터를 조회하는 기능과 관계 없음 @RequestParam X, @ModelAttribute X
- **HttpEntity는 응답에도 사용 가능**
    - 메시지 바디 정보 직접 반환
    - 헤더 정보 포함 가능
    - view 조회X
- HttpEntity 를 상속받은 다음 객체들도 같은 기능을 제공한다.
    - **RequestEntity**
        - HttpMethod, url 정보가 추가, 요청에서 사용
    - **ResponseEntity**
        - HTTP 상태 코드 설정 가능, 응답에서 사용
        - return new ResponseEntity<String>("Hello World", responseHeaders,
        
        HttpStatus.CREATED)
        
- **@RequestBody**
    - @RequestBody 를 사용하면 HTTP 메시지 바디 정보를 편리하게 조회할 수 있다. 참고로 헤더 정보가 필요하다면 HttpEntity 를 사용하거나 @RequestHeader 를 사용하면 된다.
    - 이렇게 메시지 바디를 직접 조회하는 기능은 요청 파라미터를 조회하는 @RequestParam ,
    @ModelAttribute 와는 전혀 관계가 없다.
- **요청 파라미터 vs HTTP 메시지 바디**
    - 요청 파라미터를 조회하는 기능: @RequestParam , @ModelAttribute
    - HTTP 메시지 바디를 직접 조회하는 기능: @RequestBody
- **@ResponseBody**
    - @ResponseBody 를 사용하면 응답 결과를 HTTP 메시지 바디에 직접 담아서 전달할 수 있다.
    물론 이 경우에도 view를 사용하지 않는다.

**HTTP 요청 메시지 - JSON**

- HttpServletRequest를 사용해서 직접 HTTP 메시지 바디에서 데이터를 읽어와서, 문자로 변환한다.
- 문자로 된 JSON 데이터를 Jackson 라이브러리인 objectMapper 를 사용해서 자바 객체로 변환한다.
- **@RequestBody 객체 파라미터**
    - @RequestBody HelloData data
    - @RequestBody 에 직접 만든 객체를 지정할 수 있다.
- HttpEntity , @RequestBody 를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다.
- **@RequestBody는 생략 불가능**

**HTTP 응답 - 정적 리소스, 뷰 템플릿**

- 정적 리소스
    - 예) 웹 브라우저에 정적인 HTML, css, js을 제공할 때는, **정적 리소스**를 사용한다.
- 뷰 템플릿 사용
    - 예) 웹 브라우저에 동적인 HTML을 제공할 때는 뷰 템플릿을 사용한다.
- HTTP 메시지 사용
    - HTTP API를 제공하는 경우에는 HTML이 아니라 데이터를 전달해야 하므로, HTTP 메시지 바디에
    JSON 같은 형식으로 데이터를 실어 보낸다.

**HTTP 메시지 컨버터**

- @ResponseBody 를 사용
    - HTTP의 BODY에 문자 내용을 직접 반환
    - viewResolver 대신에 HttpMessageConverter 가 동작
    - 기본 문자처리: StringHttpMessageConverter
    - 기본 객체처리: MappingJackson2HttpMessageConverter
    - byte 처리 등등 기타 여러 HttpMessageConverter가 기본으로 등록되어 있음
- **스프링 MVC는 다음의 경우에 HTTP 메시지 컨버터를 적용한다.**
    - HTTP 요청: @RequestBody , HttpEntity(RequestEntity)
    - HTTP 응답: @ResponseBody , HttpEntity(ResponseEntity)
- 스프링 부트는 다양한 메시지 컨버터를 제공하는데, 대상 클래스 타입과 미디어 타입 둘을 체크해서 사용여부를 결정한다. 만약 만족하지 않으면 다음 메시지 컨버터로 우선순위가 넘어간다.

</div>
</details>

<details>
<summary>로그</summary>
<div markdown="1">       

**로그 레벨**

- TRACE > DEBUG > INFO > WARN > ERROR
- 개발 서버는 debug 출력
- 운영 서버는 info 출력

**로그 레벨 설정**

- application.properties

```
#전체 로그 레벨 설정(기본 info)
logging.level.root=info

#hello.springmvc 패키지와 그 하위 로그 레벨 설정
logging.level.hello.springmvc=debug
```

**올바른 로그 사용법**

- log.debug("data="+data)
    - 로그 출력 레벨을 info로 설정해도 해당 코드에 있는 "data="+data가 실제 실행이 되어 버린다. 결과적으로 문자 더하기 연산이 발생한다.
- log.debug("data={}", data)
    - 로그 출력 레벨을 info로 설정하면 아무일도 발생하지 않는다. 따라서 앞과 같은 의미없는 연산이 발생하지 않는다.

**로그 사용시 장점**

- 쓰레드 정보, 클래스 이름 같은 부가 정보를 함께 볼 수 있고, 출력 모양을 조정할 수 있다.
- 로그 레벨에 따라 개발 서버에서는 모든 로그를 출력하고, 운영서버에서는 출력하지 않는 등 로그를 상황에 맞게 조절할 수 있다.
- 시스템 아웃 콘솔에만 출력하는 것이 아니라, 파일이나 네트워크 등, 로그를 별도의 위치에 남길 수 있다. 특히 파일로 남길 때는 일별, 특정 용량에 따라 로그를 분할하는 것도 가능하다.
- 성능도 일반 System.out보다 좋다. (내부 버퍼링, 멀티 쓰레드 등등) 그래서 실무에서는 꼭 로그를 사용해야 한다.


</div>
</details>

<details>
<summary>타임리프</summary>
<div markdown="1">       

**타임리프 사용 선언**

- <html xmlns:th="http://www.thymeleaf.org">

**속성 변경 - th:href**

- th:href="@{/css/bootstrap.min.css}"
- href="value1" 을 th:href="value2" 의 값으로 변경한다.
- 타임리프 뷰 템플릿을 거치게 되면 원래 값을 th:xxx 값으로 변경한다. 만약 값이 없다면 새로 생성한다.
- HTML을 그대로 볼 때는 href 속성이 사용되고, 뷰 템플릿을 거치면 th:href 의 값이 href 로
대체되면서 동적으로 변경할 수 있다.
- 대부분의 HTML 속성을 th:xxx 로 변경할 수 있다.

**타임리프 핵심**

- 핵심은 th:xxx 가 붙은 부분은 서버사이드에서 렌더링 되고, 기존 것을 대체한다. th:xxx 이 없으면 기존
html의 xxx 속성이 그대로 사용된다.
- HTML을 파일로 직접 열었을 때, th:xxx 가 있어도 웹 브라우저는 th: 속성을 알지 못하므로 무시한다.
- 따라서 HTML을 파일 보기를 유지하면서 템플릿 기능도 할 수 있다.

**URL 링크 표현식 - @{...}**,

- th:href="@{/css/bootstrap.min.css}"
- @{...} : 타임리프는 URL 링크를 사용하는 경우 @{...} 를 사용한다. 이것을 URL 링크 표현식이라 한다.
- URL 링크 표현식을 사용하면 서블릿 컨텍스트를 자동으로 포함한다.

**상품 등록 폼으로 이동**

- **속성 변경 - th:onclick**
- onclick="location.href='addForm.html'"
- th:onclick="|location.href='@{/basic/items/add}'|"

**리터럴 대체 - |...|**

- |...| :이렇게 사용한다.
- 타임리프에서 문자와 표현식 등은 분리되어 있기 때문에 더해서 사용해야 한다.
    - <span th:text="'Welcome to our application, ' + ${user.name} + '!'">
- 다음과 같이 리터럴 대체 문법을 사용하면, 더하기 없이 편리하게 사용할 수 있다.
    - <span th:text="|Welcome to our application, ${user.name}!|">
- 결과를 다음과 같이 만들어야 하는데
    - location.href='/basic/items/add'
- 그냥 사용하면 문자와 표현식을 각각 따로 더해서 사용해야 하므로 다음과 같이 복잡해진다.
    - th:onclick="'location.href=' + '\'' + @{/basic/items/add} + '\''"
- 리터럴 대체 문법을 사용하면 다음과 같이 편리하게 사용할 수 있다.
    - th:onclick="|location.href='@{/basic/items/add}'|"

**반복 출력 - th:each**

- <tr th:each="item : ${items}">
- 반복은 th:each 를 사용한다. 이렇게 하면 모델에 포함된 items 컬렉션 데이터가 item 변수에 하나씩
포함되고, 반복문 안에서 item 변수를 사용할 수 있다.
- 컬렉션의 수 만큼 <tr>..</tr> 이 하위 테그를 포함해서 생성된다.

**변수 표현식 - ${...}**

- <td th:text="${item.price}">10000</td>
- 모델에 포함된 값이나, 타임리프 변수로 선언한 값을 조회할 수 있다.
- 프로퍼티 접근법을 사용한다. ( item.getPrice() )

**내용 변경 - th:text**

- <td th:text="${item.price}">10000</td>
- 내용의 값을 th:text 의 값으로 변경한다.
- 여기서는 10000을 ${item.price} 의 값으로 변경한다.

**URL 링크 표현식2 - @{...}**

- th:href="@{/basic/items/{itemId}(itemId=${item.id})}"
- 상품 ID를 선택하는 링크를 확인해보자.
- URL 링크 표현식을 사용하면 경로를 템플릿처럼 편리하게 사용할 수 있다.
- 경로 변수( {itemId} ) 뿐만 아니라 쿼리 파라미터도 생성한다.
- 예) th:href="@{/basic/items/{itemId}(itemId=${item.id}, query='test')}"
    - 생성 링크: http://localhost:8080/basic/items/1?query=test

**URL 링크 간단히**

- th:href="@{|/basic/items/${item.id}|}"
- 상품 이름을 선택하는 링크를 확인해보자.
- 리터럴 대체 문법을 활용해서 간단히 사용할 수도 있다.

**참고**

- 타임리프는 순수 HTML을 파일을 웹 브라우저에서 열어도 내용을 확인할 수 있고, 서버를 통해 뷰 템플릿을 거치면 동적으로 변경된 결과를 확인할 수 있다. JSP를 생각해보면, JSP 파일은 웹 브라우저에서 그냥 열면 JSP 소스코드와 HTML이 뒤죽박죽 되어서 정상적인 확인이 불가능하다. 오직 서버를 통해서 JSP를 열어야 한다.
- 이렇게 **순수 HTML을 그대로 유지하면서 뷰 템플릿도 사용할 수 있는 타임리프의 특징을 네츄럴 템플릿**
(natural templates)이라 한다.

</div>
</details>
