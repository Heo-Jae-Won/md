## <span style="color:#802548">_1.코드를 잘 쓰는 방법_</span>
- (1) 정의하고 할당한 변수는 최대한 빠르게 사용한다. 해당 local을 벗어나게 되면 파악하기가 어려워진다.
  - 특히 map 변수는 최악이다. map 변수를 최대한 멀리하라
  - 쓸거면 매번 다른 param을 받는 것을 명확하게 보여주기 위해 다른 map을 new로 생성하자

- (2) 중첩 if문을 최대한 자제하자
  - 중첩 if문을 1번은 쓸 수 있다. 그러나 그 한번을 쓰게 되면 그 다음 유지보수 개발자는 그 if문 안에 또 if문을 만들것이다.
  - early return이 가능하다면 early return을 하자.
  - 마찬가지 이유로 복잡한 switch case를 자제해야 한다.

- (3) method argument를 여러개 두지 않는다.
  - STM은 최대 6개까지만 기억한다. argument는 3개 아래로 만드는 게 좋다.
  - 많이 만들어야 한다면 class로 묶어 chunk로 만들자.

- (4) class의 SRP 원칙을 따르자.
  - 너무 많은 책임을 갖고 있는 class를 개발자는 기억할 수 없다.

- (5) 긴 method를 만들지 않는다.
  - method가 길어지면 개발자가 파악하는 데 한계가 있다.
  - private method로 빼고서 javadoc을 잘 달아두는 게 좋은 선태이다.

## <span style="color:#802548">_2.중복코드 제거_</span>
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

## <span style="color:#802548">_3.특정 방식으로 구현한 이유에 대해 comment_</span>
- 특정 방식으로 구현한 이유에 대해서는 코드로만 나타낼 수 없다.
- 코드만이 문서가 아니다. 이럴 땐 주석을 달자
```js
const originalAccounts = '${US_BRK_LIST}';
const accounts = '${US_BRK_LIST}' ? 'US_BRK_LIST' : '[]';
const list = JSON.parse(accounts);
```

- 그냥 위처럼 해놓으면 왜 저런 식으로 구현했는지 알기 어렵다.
- 아래처럼 왜 코드를 아래처럼 구현했는지 주석으로 명시해두면 history를 알 수 있다.
```js
/**
 * ""가 아니라 ''인 이유는 Spring에서 넘어올 때 ""를 달아 넘어오므로 다시 ""를 달면 parsing 불가
 * 문자열로 감싼 이유는 US_BRK_LIST가 넘어오지 않는 경우, parsing 불가
 * '[]'로 준 이유는 json parsing을 하려면 string처리해야 하기 때문..
 */
const originalAccounts = '${US_BRK_LIST}';
const accounts = '${US_BRK_LIST}' ? 'US_BRK_LIST' : '[]';
const list = JSON.parse(accounts);
```
