## <span style="color:#802548">_중복코드 제거_</span>
- 아래와 같은 if문이 여러군데 중복된다면 한군데만 고치면 if문을 사용하는 다른 곳에서는 빵꾸가 날 수 있다.
```java
// 직원에게 복지 혜택을 받을 자격이 있는지 검사한다.
if ((employee.flgas && HOURLY_FLAG) && (employee.age > 65))
```

- 따라서 아래와 같이 class안에 method로 묶어주자.
- 만들어둔 isEligibleForFullBenefits logic을 이제 계속 활용하면 된다.
- if문은 Employee class 안으로 encapsulation된다.
- 해당 방식은 DDD에선 가능한데, mybatis에서는 어려울 수도 있다. 
  - 특히 dto class도 없는 곳에선... 불가능하다.

```java
@Data
public class Employee {
    private boolean flags;
    private int age;

    public boolean isEligibleForFullBenefits() {
        if( (this.flags && HOURLY_FLAG) && (this.age > 65) ) {
            return true;
        } 

        return false;
    }

}

public class EntryController {

    @GetMapping("/employees/{employeeId}")
    public EmployeeDto checkDuplication(@PathVariable String employeeId) {
        Employee employee = EmployeeService.retreiveEmployee(employeeId);
        if (emplyee.isEligibleForFullBenefits()) {
            //business logic
        }
    }
}
```


- class를 못쓰는 js convention이라면 어떤 의미인지 알기 쉽게 named boolean 변수로라도 만들어주자.
```js
const isEligibleForFullBenefits = ((employee.flgas && HOURLY_FLAG) && (employee.age > 65));
```




## <span style="color:#802548">_API 문서화_</span>
- 코드 문서화의 경우, 공개 API에 관해서만 진행한다.
- 또한 내부 동작방식을 설명하면 안 된다.
- 반드시 3인칭으로 진행한다.
  - gets the label (o)
  - get the label (x)
- 메소드 설명은 동사구로 시작한다.
  - gets the label of this button

- 클래스/인터페이스/필드는 주어 생략
  - A button label(o)
  - This field is button label (x)
- 현재 클레스에서 생성된 객체를 지칭할 때는 the 대신 this
  - gets the toolkit for this componenet (o)
  - gets the toolkit for  the component (x)



- 잘쓴 javadoc API문서의 사례는 아래와 같다.
```java
/**
* Registers the text to display in a tooltip.
* This text displays when the cursor lingers
* over the component.
*
*
* @param text the string to display. If the text is
*                    null, the tool tip is turned off for
*                    this component.
*
*/
```
