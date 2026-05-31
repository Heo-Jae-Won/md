
## <span style="color:#802548">_input select_</span>

```html
<div class="form-group">
	<label for="chickenType">치킨 종류:</label>
	<select id="chickenType" name="chickenType" required>
		<option value="">-- 치킨 종류를 선택하세요 --</option>
		<option value="18000">후라이드 치킨 (18,000원)</option>
		<option value="19000">양념 치킨 (19,000원)</option>
		<option value="20000">반반 치킨 (20,000원)</option>
	</select>
</div>
```

- select tag에서 선택된 값을 가져오려면 options 중에서 선택된 index를 선택해야한다.

```js
const chickenTypeDom = document.querySelector("#chickenType");
const chickenTypeValue = chickenTypeDom.options[chickenTypeDom.selectedIndex].value;
```


## <span style="color:#802548">_input check_</span>

- all checkbox와 개별 checkbox를 조절해보자. 우선 html은 아래와 같다.

```html
<div class="form-group">
  <label>추가 메뉴:</label>
  <div>
    <input type="checkbox" id="extraAll" name="extraAll" value="10000">
    <label class="option-label" for="extraAll">전부 추가 (+10,000원)</label>
  </div>
  <div>
    <input type="checkbox" id="extraRadish" name="extraOption" value="1000">
    <label class="option-label" for="extraRadish">무 추가 (+1,000원)</label>
  </div>
  <div>
    <input type="checkbox" id="extraCola" name="extraOption" value="2000">
    <label class="option-label" for="extraCola">콜라 추가 (+2,000원)</label>
  </div>
  <div>
    <input type="checkbox" id="extraFries" name="extraOption" value="3000">
    <label class="option-label" for="extraFries">감자튀김 (+3,000원)</label>
  </div>
  <div>
    <input type="checkbox" id="extraSotteok" name="extraOption" value="4000">
    <label class="option-label" for="extraSotteok">소떡소떡 (+4,000원)</label>
  </div>
</div>
```

- check를 확인하는 속성은 checked다.
- checkbox는 name을 같게 주기 때문에, name으로 접근하고, checkbox가 복수개이기 때문에, collection으로 생각해야 한다.
- 따라서 개별 checkbox마다 eventListener를 달아줘야 한다.

```js
document.querySelector("#extraAll").addEventListener("click", function() {
	const checkAllButton = document.querySelector("#extraAll");
	if (checkAllButton.checked) {
		for (const element of document.querySelectorAll("input[name='extraOption']")) {
			element.checked = true;
		}
	} else {
		for (const element of document.querySelectorAll("input[name='extraOption']")) {
			element.checked = false;
		}
	}
})
const allCheckCount = document.querySelectorAll("input[name='extraOption']").length;
for (const element of document.querySelectorAll("input[name='extraOption']")) {
	element.addEventListener('click', function() {
		const checkedCount = document.querySelectorAll("input[name='extraOption']:checked").length;
		if (allCheckCount === checkedCount) {
			document.querySelector("#extraAll").checked = true;
		} else {
			document.querySelector("#extraAll").checked = false;
		}
	})
}
```

## <span style="color:#802548">_input radio_</span>

```html
<div class="form-group">
	<label>배달 방식:</label>
	<div>
		<input type="radio" id="deliveryBasic" name="deliveryType" value="0" checked>
		<label class="option-label" for="deliveryBasic">알뜰배달 (무료)</label>
	</div>
	<div>
		<input type="radio" id="deliveryPremium" name="deliveryType" value="1000">
		<label class="option-label" for="deliveryPremium">한집배달 (+1,000원)</label>
	</div>
</div>
```

- input radio는 1개의 check된 값만 가져오면 된다.

```js
const deliveryTypeDom = document.querySelector("input[name='deliveryType']:checked").value;
```

## <span style="color:#802548">_그 외 흔한 input tag_</span>

- 그냥 input tag는 value 속성을 가져오면 된다.

```js
const chickenQuantityDom = document.querySelector("#quantity");
const chickenQuantityValue = chickenQuantityDom.value;
```


## <span style="color:#802548">_input과 인접한 label text 가져오기_</span>

- 매번 선택한 값이 바뀔 때마다 새롭게 DOM을 가져와야 한다.
- 따라서 이전에 있던 값들을 전부 지워주는 작업이 필요하다.
- 그 뒤에 추가해주는데, element가 input이기 때문에, 그 옆에 있는 label Node를 가져와야 한다.
	- 그런데 그냥 가져오면 해당 HTML 요소가 이동하는 것이다. 따라서 deepCopy로 clone된 Node를 가져와야 한다.
	- cloneNode를 false로 놓으면 가져와지지가 않는다.

```js
const extraOptionDoms = document.querySelectorAll("input[name='extraOption']:checked");
const receiptExtrasDom = document.querySelector("#receiptExtras");
while (receiptExtrasDom.hasChildNodes()) {
	receiptExtrasDom.removeChild(receiptExtrasDom.firstChild);
}
for (const element of extraOptionDoms) {
	receiptExtrasDom.appendChild(element.nextElementSibling.cloneNode(true))
}
```


## <span style="color:#802548">_내부의 text 값을 변경할 때_</span>

- textcontent는 text의 내용을 바꿀 때 사용한다.

```js
document.querySelector("#error").textContent = "치킨 종류를 선택해주세요.";
```

## <span style="color:#802548">_자식 요소 찾기에서 data attr 값 찾기_</span>

```html
 <tr th:each="item, status : ${returns}">
	<td th:attr="data-car-seq=${item.carSeq}" th:text="${item.carSeq}"></td>
	<td th:text="${item.carType}"></td>
	<!--temporals로 해야됨. dates로는 안 됨.-->
	<td th:text="${#temporals.format(item.sharingDate, 'yyyy-MM-dd HH:mm:ss')}"></td>
	<td th:text="${item.isNotReturned ? '예약 중' : '반납 완료'}"></td>
	<td>
		<th:block th:if="${item.isNotReturned == true}">
		<button type="button" class="returnButton" th:attr="data-seq=${item.orderSeq}">반납하기</button>
		</th:block>
	</td>
</tr>
```

- button을 복수개로 만들기 때문에, id가 아닌 class로 준다.
- class로 접근하여 모든 button에 event를 준다.
- eventListener 안의 this는 event가 일어난 Dom이다.
	- data attr 속성은 dataset에 담겨있다.
- 그런데 car-seq의 경우에는 버튼의 상위에 있기 때문에, 부모-부모인 tr로 갔다가 자식 요소에서 찾아야 한다.
	- data 속성은 접근시 아래와 같이 활용한다. car-seq의 경우, js에선 camelCase로 접근해야 한다.

```js
const returnButton = document.querySelectorAll(".returnButton");
for (const element of returnButton) {
	element.addEventListener("click", function() {
		const orderSeq = this.dataset.seq;
		//data 속성은camelCase로 변환됨.
		const carSeq = this.parentElement.parentElement.querySelector('[data-car-seq]').dataset.carSeq; 
	})
}
```

## <span style="color:#802548">_가져온 값을 Spring에 전달하는 fetch_</span>

- Spring에서 @ModelAttribute로 받는다면, headers가 application/x-www-form-urlencoded라서 굳이 명시할 필요가 없다.
- 대신 URLSearchParams에 담아서 보내야 한다. id가 바로 input name과 같은 역할을 하게 된다.
- fetch에선 data라는 key가 아닌 body라는 key다. 
- fetch의 경우, 처음에 response를 무조건 return한 뒤 그 return 한 값을 받아서 활용하는 방식이다.

```js
const id = document.querySelector("#user_id");
if (!id) {
	return;
}

const params = new URLSearchParams();
params.append("id", id.value);

fetch('/idCheck', {
	method:'post',
	body:params,
	headers:{
		'content-type':'application/x-www-form-urlencoded'
	}
}).then((response) => {
	return response.json();
}).then((validationResult) => {
	if (validationResult) {
		alert("사용할 수 없는 ID입니다.");
	} else {
		alert("사용할 수 있는 Id입니다.");
	}

	isDuplicated = validationResult;
}).catch((error) => {
	console.log(error);
})
```


## <span style="color:#802548">_새로운 HTML을 fetch하기_</span>

```html
<table>
        <thead>
            <tr>
                <td>작성일</td>
                <td>상태</td>
                <td>중요도</td>
                <td>분류</td>
                <td>todo</td>
                <td>삭제</td>
            </tr>
        </thead>
        <tbody id="list-data">

        </tbody>
    </table>
```

- generateHTML은 generate라는 이름에 걸맞게 요소를 추가하는 작업만 실행한다.
- class로 button을 달아주는 것을 진행하면, 똑같은 event가 중첩되게 되어버린다.
- tag는 HTML 요소의 string을 만들엏 넣으며, beforeend로 넣어야 한다.

```js
function generateHTML(element, index) {
	const dataTable = document.querySelector("#list-data");
	const tag = `
			<tr>
				<td>${element.regDate}</td>
				<td>${element.status}</td>
				<td>${element.importance}</td>
				<td>${element.categories}</td>
				<td>${element.todo}</td>
				<td><button type=button data-seqNo=${element.seqNo} id=deleteBtn${index}></td>
			</tr>
	`;
	dataTable.insertAdjacentHTML('beforeend', tag)

	const button = document.querySelector(`deleteBtn${index}`);
	button.addEventListener('click', function() {
		if(!confirm("삭제하시겠습니까?")) {
			return;
		}

		fetch(`/todo/delete?id=${index}`,{
			method:'post'
		}).then((response) => {
			return response.json();
		}).then((data) => {
			console.log(data);
		})
	}) 
}
```

- fetchList를 할 때는 무조건 이전의 모든 HTML 요소를 삭제해줘야 한다.
- 그런데 그냥 document로 가져오면 live collection이라 제대로 삭제가 안 되므로, Array.from을 통해 js 객체로 만든다.
- 그리고 모든 하위 요소를 삭제시킨다.

```js
function fetchList() {
	const children = Array.from(document.querySelector("#list-data").childNodes);  // Get all child nodes (including text nodes)
	children.forEach(child => {
		document.querySelector("#list-data").removeChild(child);  // Remove each child node
	});

	fetch('/todo/list',{
		method:'get'
	}).then((response) => {
		return response.json();
	}).then((list) => {
		let index = 0;
		for (const element of list) {
			generateHTML(element, index);
			index++;
		}
	}).catch((error) => {
		console.log(error);
	})
}
```


## <span style="color:#802548">_ts HTML manipulation_</span>

- querySelector로 할 때는 id나 class보다는 element를 주면 좋다.
- interface를 ts가 알아서 추론해주어 그 interface의 method를 쓸 수 있다.
- id나 class를 쓰면 any type이라서 generics로 덮어 써야 method를 쓸 수 있다.

```javascript
const form = document.querySelector<HTMLFormElement>('#form');
form.addEventListener("submit",function(e){//e가 뭘 의미하는 지 암. submit에 addEventLister기 때문.
    e.preventDefault();
    console.log("SUBMITTED");
})

const handleSubmit = function(e /*e: subMitEvent()로 써줘야 함. */){ //e가 뭘의미하는 지 ts가 모름. context가 없기 때문. 따라서 error
    
    e.preventDefault();
    console.log("SUBMITTED");
}
form.addEventListener("submit",handleSubmit); 
```

- localStorage는 문자열만 갖고 있다. 그래서 객체를 넣을 수가 없다.
- 그래서 stringify로 바꿔서 넣고, parse로 꺼내 쓴다.

```javascript
localStorage.setItem('todos', JSON.Stringify(toods));
```