## @RequestMapping 핸들러 매핑 
@MVC의 가장 큰 특징은 핸들러 매핑과 핸들러 어댑터의 대상이 오브젝트가 아니라 메소드라는 점이다. URL 당 하나의 컨트롤러 오브젝트가 매핑되고 하나의 메소드만 DispatcherServlet의 호출을 받을 수 있다.
DefaultAnnotationHandlerMapping이 빈으로 등록되어 있어야 한다.
