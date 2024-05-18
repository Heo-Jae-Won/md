## <span style="color:#802548">_왜 javadoc을 쓰는가_</span>

- (1)javadoc을 활용하자
- method가 무엇을 return하는지, 필요한 param은 무엇인지 명시한다
- 메소드가 어떤 일을 하는 지에 관해서도 간략하게 설명을 추가한다.
- 안 적어두면 다른 사람이 해당 method를 내부 구현까지 모두 뒤져봐야할 수도 있다.
```java
/**
 * 고객번호 획득
 * @return customerNo
 * @param customerNo
 * @param cardNo
 * 
 */
```

- js도 똑같이 jsdoc을 활용해주는 게 좋다.
```js
/**
 * 인사말을 생성합니다.
 * @param name 
 * @param title
 * @returns formatting된 인사말 
 */
```

```js
/** 특정 시간과 장소에서 수행된 측정 */
interface Measurement {
  /** 어디에서 측정되었나 */
  position: Vector3D;
  /** 언제 측정되었나? epoch에서 초 단위로 */
  time: number;
  /** 측정된 운동량 */
  momentum: Vector3D;
}

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