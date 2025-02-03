---
title:  "Spring DispatcherServlet"
excerpt: Spring DispatcherServlet
categories:
  - spring
---

기존 서블릿의 경우 요청 url당 서블릿을 생성하고 그에 맞는 컨트롤러에게 요청을 보내주는 코드를 작성해야 했다. 스프링 MVC는 모든 요청을 받아 실제 작업은 다른 컴포넌트로 위임하는 DispatcherServlet을 두어 **프론트 컨트롤러** 패턴으로 설계되어 있다. 모든 요청을 한곳에서 받아 처리한 뒤, 요청에 맞는 Handler로 요청을 Dispatch하고, 해당 Handler의 실행 결과를 Http Response 형태로 만드는 역할을 한다.  

## DispatcherServlet 동작과정

<img src="https://cys779988.github.io/assets/img/dispatcherServlet.PNG">

#### doService()
모든 요청은 적절한 HttpServletRequest, HttpServletResponse 객체를 매개변수로 받는 `doService()`를 호출하게 된다. 그 후 `doDispatch()`로 위임한다.

#### doDispatch()
사용자가 요청한 값이 `doService()`를 통해 `doDispatch()`에 도달하면 모든 요청을 각각의 알맞은 Handler에게 전달하는 역할을 수행한다.

#### getHandler()
요청을 처리할 수 있는 HandlerExecutionChain을 찾아 반환한다. HandlerExecutionChain은 Object 타입으로 전달되는 handler를 원래의 Handler 타입에 맞게 변환하여 전달한다.  
DispatcherServlet은 요청을 처리할 수 있는 Handler를 찾기 위한 HandlerMapping 객체들을 리스트 형태로 가지고 있다. DispatcherServlet의 `getHandler()`는 HandlerMapping의 `getHandler()`를 호출한다. HandlerMapping의 `getHandler()`는 request에 맞는 handler를 찾아준다. 즉, Controller의 Method를 리턴하게 된다.

#### getHandlerAdapter()
`getHandler()`에서 요청을 처리할 Handler를 얻었다면 다시 `doDispatch()`는 이 handler를 처리할 HandlerAdapter를 찾기 위해 찾아낸 handlerMethod를 매개변수로 전달하여 `getHandlerAdapter()`를 호출하게 된다.  
`getHandlerAdapter()`는 인자로 전달받은 handlerMethod의 처리를 지원하는 adapter를 찾아서 리턴하는 역할을 수행한다. 그리고 찾아낸 HandlerAdapter의 `handleIntenal()`에서 자바 리플렉션을 이용하여 handlerMethod를 실제로 실행한다.

#### processDispatchResult()
`processDispatchResult()` 안에서는 `render()`를 호출한다. `render()`는 viewResolver를 통해 view를 찾고, 해당 view를 반환받는다. 그리고 뷰를 실제로 렌더링한다.

## DispatcherServlet 구성요소

#### MultipartResolver
사용자의 파일업로드 요청에 대한 처리를 담당하는 인터페이스. 스프링 부트를 사용한다면 기본 구현체가 등록되지만 스프링 레거시에서는 기본값이 null이다.

#### LocaleResolver
요청하는 클라이언트의 Locale 정보를 파악하는 인터페이스. 기본 구현체로 AcceptHeaderLocaleResolver가 사용된다.

#### ThemeResolver
웹 애플리케이션의 테마를 변경하는 인터페이스.

#### HandlerMapping
사용자가 요청한 url을 분석하여 요청을 처리할 handler를 찾는 인터페이스. 기본적으로 DispatcherServlet에는 BeanNameUrlHandlerMapping, RequestMappingHandlerMapping 2가지가 구현체가 있다. 애너테이션 기반으로 API를 작성한다면 RequestMappingHandlerMapping가 기본 구현체가 사용된다.

#### HandlerAdapter
HandlerMapping이 찾아낸 handler를 호출하고 처리하는 인터페이스. 애너테이션 기반으로 API를 작성한다면 RequestMappingHandlerAdapter가 기본 구현체로 사용된다.

#### HandlerExceptionResolver
요청 처리 중 발생한 에러를 처리하는 인터페이스.

#### RequestToViewNameTranslator
요청을 보고 요청에 대응하는 view를 찾아내는 인터페이스. 예를 들어 handler에서 반환하는 값을 때, 요청 url `/test` 에 대응하는 `test.jsp`를 반환해주는 역할을 수행한다.

#### ViewResolver
요청에 대한 응답 view를 렌더링 하는 역할을 수행한다. 별도의 ViewResolver를 등록하지 않는다면 InternalResourceViewResolver가 기본 구현체로 사용된다.

#### FlashMapManager
FlashMap 인스턴스를 가져오고 저장하는 인터페이스. FlashMap은 주로 리다이렉션을 사용할 때 요청 매개변수를 사용하지 않고 데이터를 전달할 때 사용한다. 기본 구현체로 SessionFlashMapManager 사용된다.