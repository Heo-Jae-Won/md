## <span style="color:#802548">_1. enum class 만들기_</span>
- enum class는 상수기 때문에 compile타임에 이미 완성이 되어있어야 한다. 
- 또한 동적으로 생성될 수 없다. 따라서 new로 호출이 불가능하며 생성자도 public이나 protected를 사용할 수 없다. 
- 따라서 아래에 있는 private final String message는 외부에서 쓰이는 생성자의 인수가 아니라 내부에서 쓰는 생성자의 인수다. 
- 생성은 불가능하지만, 이미 만들어져 있는 상수를 다른 클래스에서 가져올 수는 있다. 
- 그 방법이 바로 @Getter다. 아래 exception class에서 getter를 활용하게 된다. 
```java
@Getter
@RequiredArgsConstructor
public enum ErrorEnum {

	NO_PRODUCT("존재하지 않는 상품입니다."),
	NO_CONTENT("파일이 없습니다"),
	NOT_ACCEPTED_TYPE_IMAGE("jpeg, png 확장자로 올려주세요"),
	VALIDATED_FALSE("주어진 양식에 맞춰 작성해주세요");

	private final String message;
}
```
## <span style="color:#802548">_2. Exception class 만들기_</span>
- 아래와 같이 RuntimeException을 extends한 Service용 exception을 만든다. 
- service용 Exception class는 생성될 때 인수로 enum class를 받으며, 생성될 때 enum에 들어있는 message를 가져오게 된다. 
- NOT_ACCEPTED_TYPE_IMAGE라는 상수의 경우는 "jpeg, png 확장자로 올려주세요"라는 메시지를 던지게 되는 것이다. 
```java
public class ServiceException extends RuntimeException {

	/**
	 * 
	 */
	private static final long serialVersionUID = 1L;

	public ServiceException(ErrorEnum errEnum) {
		super(errEnum.getMessage());
	}
}
```
- 소스코드로 보면 Exception처리를 상위 클래스를 생성하여 맡기는데, 그게 바로 super()다. 
- super()는 Throwable class까지 올라가며, Throwable class에서는 하위 클래스에서 넘어온 string 값을 자신의 field 값으로 집어넣는다. 
- 해당 field를 get하는 과정은 아래 ExceptionAdvice에서 이뤄진다. 
```java
   public RuntimeException(String message) {
        super(message);
    }
public Exception(String message) {
        super(message);
    }
public Throwable(String message) {
        fillInStackTrace();
        detailMessage = message;
    }
```
## <span style="color:#802548">_3. ExceptionAdvice 만들기_</span>
- @RestControllerAdvice는 자동으로 등록된다. 
- @ExceptionHandler에는 자신이 다룰 exception class를 지정한다. 
- 그리고 parameter로 해당 exception class를 받는다. 
```java
@RestControllerAdvice(annotations = RestController.class)
public class ExceptionAdvice {

@ExceptionHandler(ServiceException.class)
@ResponseStatus(code = HttpStatus.INTERNAL_SERVER_ERROR) // org.apache로 import하면 안됨.
	public ExceptionDto excepion(HttpServletRequest request, ServiceException serviceException) {
		ExceptionDto exceptionDto = new ExceptionDto();
		exceptionDto.setRequestURI(request.getRequestURI());
		exceptionDto.setMessage(serviceException.getMessage());
		return exceptionDto;
	}
}
```

- Exception class는 모두 getMessage()라는 method를 갖는데, 그 이유는 최상위인 Throwable class에 해당 method가 존재하기 때문이다. 
- 우리가 enum에서 규정했던 상수 안의 string 값이 detailMessage가 되고 우리는 그 값을 가져오는 것이다.
- 그 값을 가져와서 우리가 return하고자 하는 dto에 넣어준다. 
```java
@Data
public class ExceptionDto {
	private String message;
	private String requestURI;
}
```
- 위의 과정을 마쳤다면 이제 component를 등록시켜줘야 한다. 
- exception은 web과 관련이 있어 servlet-context.xml에 등록했다. @RestControllerAdvice가 @Component의 하위 annotation이다. boot의 경우에는 아래 과정이 필요없다. @SpringBootApplication이 component를 모두 scan하기 때문이다. 
```xml
<context:component-scan
		base-package="com.example.exception" />
```
## <span style="color:#802548">_4. Service에서 사용하기_</span>
- 그럼 아래와 같이 service class에서 사용할 수 있다. 
```java
if (!contentType.contains("image/png") && !contentType.contains("image/jpeg")) {
			throw new ServiceException(ErrorEnum.NOT_ACCEPTED_TYPE_IMAGE);
		}
```
- 그럼 아래와 같이 ExceptionHandlerExceptionResolver를 통해서 잘 나오게 된다.  
- dispatcherServlet의 HandlerExceptionResolver에 처리하게 된다. 
- 만약 ExceptionHandler가 제대로 작동하지 않는다면 dispatcherServlet의 DefaultHandlerExceptionResolver에서 처리하게 된다. 
- DefaultHandlerExceptionResolver은 Spring이 제공하는 기본형이다.