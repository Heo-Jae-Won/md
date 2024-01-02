## <span style="color:#802548">_1.Dao_</span>
- dao를 만들어 db와 통신하여 쿼리문을 실행한다. 

```java
public List<BoardDTO> getList(int page, String searchType, String keyword) {
    HashMap<String, Object> map = new HashMap<>();
    map.put("start", (page - 1) * 4);
    map.put("searchType", searchType);
    map.put("keyword", keyword);
    
    return session.selectList(namespace+".list",map);
}

public int getTotal(String searchType, String keyword) {
    HashMap<String, Object> map = new HashMap<>();
    map.put("searchType", searchType);
    map.put("keyword", keyword);
    
    return session.selectOne(namespace+".total",map);
}
```
## <span style="color:#802548">_2.controller - model_</span>
- model에 jsp에 뿌릴 객체를 담아준다. 
- 딱 떨어지지 않는다면 잘리지 않게끔 페이지를 한 개 더해준다.
- 여기서는 dao에서 바로 가져왔지만, service에서 가져오는 게 정석이다.

```java
@RequestMapping("/list")
public String list(Model model, int page, String searchType, String keyword) {
    List<BoardDTO> list = boardDao.getList(page, searchType, keyword);
    int last = (boardDao.getTotal(searchType,keyword) % 4) == 0 ? (boardDao.getTotal(searchType,keyword) / 4) :(boardDao.getTotal(searchType,keyword) / 4) + 1;
    model.addAttribute("list", list);
    model.addAttribute("page", page);
    model.addAttribute("keyword", keyword);
    model.addAttribute("num", 6);
    model.addAttribute("last", last);
    return "board/list";
}
```
## <span style="color:#802548">_3. jsp_</span>

​

- controller에서 담은 값을 c태그를 활용해 jsp에서 load한다.
- 이 때 조건은 실제 값이 존재해야 한다는 것이다. 
- 아래에서는 page를 보여준다.
```html
<c:if test="${last == 0}">
				<span>검색된 자료가 없습니다. </span>
</c:if>
<c:if test="${last != 0}">
				<button id="prev" type="submit"
					<c:out value="${page==1?'disabled':''}"/>>이전</button>
				<span> <c:out value="${page}/${last}" /> </span>
				<button id="next" type="submit"
					<c:out value="${page==last?'disabled':''}"/>>다음</button>
</c:if>
```
- 아래에서는 page에 따라 달라지는 값을 보여준다.
- items 안에는 controller에서 담은 model이 들어가 있으며, 이를 vo라는 변수명을 통해 property를 가져올 수 있다. 
```html
<table class="table table-bordered">
			<thead>
				<tr>
					<th scope="col">번호</th>
					<th scope="col">제목</th>
					<th scope="col">내용</th>
					<th scope="col">작성자</th>
					<th scope="col">조회수</th>
					<th scope="col">날짜</th>
					<th scope="col">이미지</th>
				</tr>
			</thead>
			<tbody>
				<c:forEach items="${list}" var="vo">
					<tr>
						<th onclick="location.href='/board/read/${vo.boardNo}'"
							scope="row"><c:out value="${vo.boardNo }" /></th>
						<td><c:out value="${vo.boardTitle}" /></td>
						<td><c:out value="${vo.boardContent}" /></td>
						<td><c:out value="${vo.boardWriter}" /></td>
						<td><c:out value="${vo.boardView}" /></td>
						<td><c:out value="${vo.boardRegisterDate}" /></td>
						<td><img src="<c:out value="${vo.boardPhoto}"/>" alt="빈 이미지"
							width="100" height="100" /></td>
					</tr>
				</c:forEach>
			</tbody>
		</table>
​

<div class="content-wrapper">
		<form name="frm">
			<input type="hidden" value="<c:out value="${page}"/>" size=2 name="page"> 
				<select name="searchType" id="searchType">
					<option>제목</option>
					<option>내용</option>
					<option>작성자</option>
					<option>제목과 내용</option>
				</select> 
			<input type="text" name="keyword" id="keyword" value="${keyword}">
			<c:if test="${last == 0}">
				<span>검색된 자료가 없습니다. </span>
			</c:if>

			<c:if test="${last != 0}">
				<button id="prev" type="submit"
					<c:out value="${page==1?'disabled':''}"/>>이전</button>
				<span> <c:out value="${page}/${last}" /> </span>
				<button id="next" type="submit"
					<c:out value="${page==last?'disabled':''}"/>>다음</button>
			</c:if>

		</form>
		<table class="table table-bordered">
			<thead>
				<tr>
					<th scope="col">번호</th>
					<th scope="col">제목</th>
					<th scope="col">내용</th>
					<th scope="col">작성자</th>
					<th scope="col">조회수</th>
					<th scope="col">날짜</th>
					<th scope="col">이미지</th>
				</tr>
			</thead>
			<tbody>
				<c:forEach items="${list}" var="vo">
					<tr>
						<th onclick="location.href='/board/read/${vo.boardNo}'"
							scope="row"><c:out value="${vo.boardNo }" /></th>
						<td><c:out value="${vo.boardTitle}" /></td>
						<td><c:out value="${vo.boardContent}" /></td>
						<td><c:out value="${vo.boardWriter}" /></td>
						<td><c:out value="${vo.boardView}" /></td>
						<td><c:out value="${vo.boardRegisterDate}" /></td>
						<td><img src="<c:out value="${vo.boardPhoto}"/>" alt="빈 이미지"
							width="100" height="100" /></td>
					</tr>
				</c:forEach>
			</tbody>
		</table>
	</div>
```
## <span style="color:#802548">_4. js_</span>
- 이전과 다음 버튼을 클릭 시에 페이지를 증감시킨다.
- 엔터를 누르게되면 검색을 실행한다.

```js
let page = parseInt($(frm.page).val()) || 1;

$("#prev").on("click", function() {
	--page;
	page = $(frm.page).val(page);

});

$("#next").on("click", function() {
	++page;
	page = $(frm.page).val(page);
});

$("#keyword").on("keydown", function(event) {
	if (event.keyCode === 13) {
		frm.submit();
	}
})
```