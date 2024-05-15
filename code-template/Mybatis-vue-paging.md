## <span style="color:#802548">_1. Dto_</span>
- paging을 위해 front에 보내는 정보는 아래와 같다.
  - list
  - lastPage
```java
@Data
public class BoardListResponse {
	List<BoardDto>boardList;
	int boardListLastPageNumber;
}
```
## <span style="color:#802548">_2. Service_</span>
- list와 lastPageNumber를 front에 보내주기 위해 서비스 로직이다. 
```java
public List<BoardDto> getList(int page, String searchType, String keyword) {
		int start = (page - 1) * 6;
		List<BoardDto> list = boardMapper.list(start, searchType, keyword);
		return list;
	}
public int getLast(String searchType, String keyword) {
		int last = (boardMapper.total(searchType, keyword) % 6) == 0 ? (boardMapper.total(searchType, keyword) / 6)
				: (boardMapper.total(searchType, keyword) / 6) + 1;

		return last;
	}
```
## <span style="color:#802548">_3. Mapper Interface_</span>
- 그럼 아래와 같이 service에서 활용할 mapper interface를 만든다.
```java
@Mapper
public interface BoardMapper {

	List<BoardDto> list(@Param("start") int page, @Param("searchType") String searchType, @Param("keyword")String keyword);

	int total(@Param("searchType") String searchType, @Param("keyword")String keyword);

	BoardDto read(int boardNo);

	void insert(BoardDto boardDto);
	
	void delete(int boardNo);
	
	void update(BoardDto boardDto);
	
	void increaseView(int boardNo);
}
```
## <span style="color:#802548">_4.Sql Mapper XML_</span>

- 위의 mapper interface가 수행할 query를 담은 xml을 만든다.
```java
<select id="list" resultMap="boardInfo">
		select board_no, board_title, board_content,
		board_writer, board_register_date, board_photo, board_view
		from board
		<if test='searchType != null and searchType.equals("제목")'>
			WHERE board_title LIKE concat('%', #{keyword}, '%')
		</if>
		<if test='searchType != null and searchType.equals("내용")'>
			WHERE board_content LIKE concat('%', #{keyword}, '%')
		</if>
		<if test='searchType != null and searchType.equals("작성자")'>
			WHERE board_writer LIKE concat('%', #{keyword}, '%')
		</if>
		<if test='searchType != null and searchType.equals("제목과 내용")'>
			WHERE board_title LIKE concat('%', #{keyword}, '%') 
			or board_content LIKE concat('%', #{keyword}, '%') 
		</if>
		limit #{start},6
	</select>

	<select id="total" resultType="int">
		select count(board_no)
		from board
		<if test='searchType != null and searchType.equals("제목")'>
			WHERE board_title LIKE concat('%', #{keyword}, '%')
		</if>
		<if test='searchType != null and searchType.equals("내용")'>
			WHERE board_content LIKE concat('%', #{keyword}, '%')
		</if>
		<if test='searchType != null and searchType.equals("작성자")'>
			WHERE board_writer LIKE concat('%', #{keyword}, '%')
		</if>
		<if test='searchType != null and searchType.equals("제목과 내용")'>
			WHERE board_title LIKE concat('%', #{keyword}, '%') 
			or board_content LIKE concat('%', #{keyword}, '%') 
		</if>
	</select>
```
## <span style="color:#802548">_5.Controller_</span>
- 프론트에 json 형식으로 보낸다.
```java
@RequestMapping(value = "/api/boards", method = RequestMethod.GET)
	public ResponseEntity<String> fetchList(int page, String searchType, String keyword,
			HttpServletResponse response) throws JsonProcessingException {
		BoardListResponse boardListResponse = new BoardListResponse();
		boardListResponse.setBoardList(boardService.getList(page, searchType, keyword));
		boardListResponse.setBoardListLastPageNumber(boardService.getLast(searchType, keyword));

		return new ResponseEntity<String>(new ObjectMapper().writeValueAsString(boardListResponse), HttpStatus.OK);
	}
```

- Mapper를 인식시켜주려면 application을 run하는 main method class를 찾자.
- 그리고 @MapperScan을 달아주자.

## <span style="color:#802548">_6. Axios_</span>
- 백엔드에 api를 보내기 위해 axios를 활용한다. api를 통해 목록과 마지막 페이지 정보를 받아 활용한다. 

```javascript
const pageArray = ref<Array<BoardResponse>>([]);
const page = ref(1);
const last = ref(1);
const fetchBoardList = async () => {
    await getBoardList(page.value, searchType.value, keyword.value).then(res => {
        pageArray.value = res.data.boardList;
        last.value = res.data.boardListLastPageNumber;
    });
};
```
- dom이 생성되고 data가 load되기 전인 시점(onMounted)에 위의 axios 함수를 호출하여 data를 받아온다.
```javascript
onMounted(() => {
      getBoardList(1, '제목', '').then(res => {
        pageArray.value = res.data.boardList;
        last.value = res.data.boardListTotal;
      });
    });
```
## <span style="color:#802548">_7. v-for를 이용해 목록을 render_</span>
- 아래와 같이 v-for를 활용하여 목록을 rendering한다.
```java
 <tr v-for="board in pageArray" :key="board.boardNo">
        <td @click="$router.push(`/board/read/${board.boardNo}`)">{{ board.boardNo }}</td>
        <td>{{ board.boardTitle }}</td>
        <td>{{ board.boardContent }}</td>
        <td>{{ board.boardWriter }}</td>
        <td>{{ board.boardView }}</td>
        <td>{{ board.boardRegisterDate }}</td>
        <img :src="'/upload/' + board.boardPhoto" alt="빈 이미지" />
      </tr>
```
​

## <span style="color:#802548">_8. v-on을 이용해 paging_</span> 
- v-on(:)을 활용하여 특정 조건에 disabled 특성을 부여한다.
```javascript
   <v-btn :disabled="page <= 1" @click="prevPage()" class="page-btn">이전</v-btn>
      <span class="page-count">{{ page }} / {{ last }} 페이지</span>
   <v-btn :disabled="page >= last" @click="nextPage()" class="page-btn">다음</v-btn>
```
- click 시에 page가 바뀌면서 axios가 api를 호출하고, vue가 변경점을 감지하여 다시 render한다. 
```javascript
const nextPage = () => {
      page.value += 1;
      fetchBoardList();
    };

    const prevPage = () => {
      page.value -= 1;
      fetchBoardList();
    };
​

 <v-select
      v-model="searchType"
      block
      class="w-25"
      label="검색타입"
      :items="['제목', '내용', '작성자', '제목과 내용']" />
    <v-text-field v-model="keyword" @keyup.enter="fetchFilterdBoardList()" />
    <v-table theme="dark">
      <tr>
        <th>번호</th>
        <th>제목</th>
        <th>내용</th>
        <th>작성자</th>
        <th>조회수</th>
        <th>날짜</th>
        <th>이미지</th>
      </tr>
      <tr v-for="board in pageArray" :key="board.boardNo">
        <td @click="$router.push(`/board/read/${board.boardNo}`)">{{ board.boardNo }}</td>
        <td>{{ board.boardTitle }}</td>
        <td>{{ board.boardContent }}</td>
        <td>{{ board.boardWriter }}</td>
        <td>{{ board.boardView }}</td>
        <td>{{ board.boardRegisterDate }}</td>
        <img :src="'/upload/' + board.boardPhoto" alt="빈 이미지" />
      </tr>
    </v-table>
    <div class="btn-cover">
      <v-btn :disabled="page <= 1" @click="prevPage()" class="page-btn">이전</v-btn>
      <span class="page-count">{{ page }} / {{ last }} 페이지</span>
      <v-btn :disabled="page >= last" @click="nextPage()" class="page-btn">다음</v-btn>
    </div>
```