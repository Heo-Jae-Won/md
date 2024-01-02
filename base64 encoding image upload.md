## <span style="color:#802548">_1. data_</span>
​

- fileData를 받아오는 것과, 이를 base64string으로 만드는 것을 위해 두 가지 변수를 선언한다.

```javascript
const fileData = ref<Blob | null>(null);
const base64Data = ref<string | ArrayBuffer | null>(null);
```
- fileChange 함수는 event를 받아서 fileData의 값을 바꿔준다.

```html
  <v-col cols="12" md="4">
    이미지:<v-file-input @change="onFileChange($event)" :multiple="false" class="h-50" />
  </v-col>
  ```
```javascript
const onFileChange = ($event: Event) => {
    const target = $event.target as HTMLInputElement;

    if (target && target.files) {
        fileData.value = target.files[0];
    }
}
```
- 그럼 아래와 같이 reader를 통해 읽어 base64encoding 값으로 전환된 file 값을 가져올 수 있다.
```javascript
const file = fileData.value;
reader.readAsDataURL(file as Blob);
reader.onload = function () {
    base64Data.value = reader.result;
};
const fileData = ref<Blob | null>(null);
const base64Data = ref<string | ArrayBuffer | null>(null);

const onFileChange = ($event: Event) => {
      const target = $event.target as HTMLInputElement;

      if (target && target.files) {
        fileData.value = target.files[0];
      }

      const file = fileData.value;
      reader.readAsDataURL(file as Blob);
      reader.onload = function () {
        base64Data.value = reader.result;
      };
    };
```
## <span style="color:#802548">_2. axios_</span>
- type은 file만 특별하게 union type인데, 그 이유는 위에서 reader.result가 아래와 같은 type을 가져야 하기 때문이다.

```javascript
export class BoardRequest {}
export interface BoardRequest {
  boardTitle: string;
  boardContent: string;
  boardWriter: string;
  file: string | ArrayBuffer | null;
}
```
그럼 아래와 같이 axios로 보내주면 된다.
```javascript
const board = {
    boardTitle: boardInfo.value.boardTitle,
    boardContent: boardInfo.value?.boardContent,
    boardWriter: boardInfo.value.boardWriter,
    file: base64Data.value,
};

export const onInsertBoardInfo = (data: BoardRequest) => {
  return instance({
    url: '/boards',
    method: 'post',
    data: data,
  });
};
```
## <span style="color:#802548">_3. Dto_</span>
- 이제 backend에서 받아야 한다. 
- bas64로 encoding 된 image를 file이라는 data에 담아서 보냈으므로 file 이라는 string type의 field를 추가했다. 

```java
@Data
public class BoardDto {

	private int boardNo;
	private int boardView;
	@NotBlank
	@Size(max = 50)
	private String boardTitle;
	@NotBlank
	@Size(max = 300)
	private String boardContent;
	@NotBlank
	@Size(max = 20)
	private String boardWriter;
	private String boardPhoto;
	@JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd", timezone = "Asia/Seoul")
	private Date boardRegisterDate;
	private String file;
}
```
## <span style="color:#802548">_4. Controller_</span>

​

- Controller는 기존과 달리 MultiPartFile이 사라진 모습이다.
- 이제 @RequestBody에 전부 담아갈 수 있다. content-type도 application/json으로 보내야 한다.

```java
@RequestMapping(value = "/api/boards", method = RequestMethod.POST)
	public ResponseEntity<String> insert(@Valid @RequestBody BoardDto insertDto)
			throws ServiceException {

		boardService.insertBoardInfo(insertDto);

		return new ResponseEntity<String>(HttpStatus.OK);
	}
```
## <span style="color:#802548">_5. Service_</span>
- fileName은 아래와 같이 해줬다. 
- insertDto.getBoardTitle() 대신 랜덤난수 숫자(salt)를 추가해주는 것도 좋을 것 같다.
- 그냥 LocalDateTime으로 하면 아래와 같은 오류가 나서 콜론을 모두 치환해줬다.

```java
String fileName = LocalDateTime.now()
				.format(DateTimeFormatter.ofPattern("uuuu-MM-dd_HH-mm-ss"))
				    .replace(":", "-")
				        .replace(".", "") + insertDto.getBoardTitle() + ".jpg";
```
- base64로 받아온 것에서 +나 /를 모두 치환해준다.
  - 아니면 base64로 encoding된 file을 decode할 수가 없다. 
- 처음에는 contentType이 나오고, 이후부터 실제 binary 값이 나오기 때문에 substring()으로 ,이후부터 값을 decode한다.

```java
byte[] decodedBytes = Base64.getUrlDecoder()
					.decode(insertDto.getFile().substring(insertDto.getFile().lastIndexOf(",") + 1)
                        .replace('+', '-')
							.replace('/', '_'));
```
- 그 다음 확장자명을 알아내기 위해 아래와 같이 처음 contentType만 받아서 검사한다.
- chatGPT가 알려준 코드인데 잘 작동한다. 근데 무슨 의미인지 잘 모르겠어서 나중에 알아봐야할 거 같다.

​

- 아래 구문은 처음 들어오는 decoding 된 btye 중 4개를 16진수 string으로 바꾼다는 것이다. 
```java
for (int i = 0; i < 4; i++) { // Consider first 4 bytes as file signature
    sb.append(String.format("%02X", decodedBytes[i]));
}
```
​

- 그렇게 파일 시그니처를 알게되면 그게 확장자와 대응되므로 switch case로 경우를 나눠서 확장자명을 부여할 수 있다.

http://forensic-proof.com/archives/300
```java
StringBuilder sb = new StringBuilder();
			for (int i = 0; i < 4; i++) { // Consider first 4 bytes as file signature
				sb.append(String.format("%02X", decodedBytes[i]));
			}

			switch (sb.toString()) {
				case "FFD8FFE0": // JPEG image
				case "FFD8FFE1": // JFIF image
					fileExtension = ".jpg";
					break;
				case "89504E47": // PNG image
					fileExtension = ".png";
					break;
				case "47494638": // GIF image
					fileExtension = ".gif";
					break;
				default: // Unknown file type
					fileExtension = "";
					break;
			}
```

- 이제 실제 확장자를 검사해서 jpg나 png일 때만 file을 write해주면 된다. 
```java
if (fileExtension == ".jpg" || fileExtension == ".png") {
    File file = new File("/upload/" + fileName + fileExtension);
    FileOutputStream fileOutputStream = new FileOutputStream(file);
    fileOutputStream.write(decodedBytes);
    fileOutputStream.close();
} else {
    throw new ServiceException("jpg만 됨 ㅇㅇ;");
}
```
- 아래와 같이 SpringFramework의 StringUtils class를 활용할 수도 있다. 
- unknown file type의 경우에는 fileExtension이 ""으로 빈칸임을 이용한 것이다. 

```java
if (StringUtils.hasText(fileExtension)) {
    File file = new File("/upload/" + fileName + fileExtension);
    FileOutputStream fileOutputStream = new FileOutputStream(file);
    fileOutputStream.write(decodedBytes);
    fileOutputStream.close();
} else {
    throw new ServiceException("jpg만 됨 ㅇㅇ;");
}
```

- 그럼 아래와 같이 jpg나 png가 아니면 에러가 뜬다.


- 반면에 잘 등록되면  아래와 같이 폴더에 이미지가 삽입된다.
```java
public void insertBoardInfo(BoardDto insertDto) throws ServiceException {

		String fileName = LocalDateTime.now()
				.format(DateTimeFormatter.ofPattern("uuuu-MM-dd_HH-mm-ss"))
				    .replace(":", "-")
				        .replace(".", "") + insertDto.getBoardTitle() + ".jpg";

		String fileExtension = "";
		try {
			// user의 DeskTop이 home directory
			byte[] decodedBytes = Base64.getUrlDecoder()
					.decode(insertDto.getFile().substring(insertDto.getFile().lastIndexOf(",") +
							1).replace('+', '-')
							.replace('/', '_'));
			StringBuilder sb = new StringBuilder();
			for (int i = 0; i < 4; i++) { // Consider first 4 bytes as file signature
				sb.append(String.format("%02X", decodedBytes[i]));
			}

			switch (sb.toString()) {
				case "FFD8FFE0": // JPEG image
				case "FFD8FFE1": // JFIF image
					fileExtension = ".jpg";
					break;
				case "89504E47": // PNG image
					fileExtension = ".png";
					break;
				case "47494638": // GIF image
					fileExtension = ".gif";
					break;
				default: // Unknown file type
					fileExtension = "";
					break;
			}

			if (fileExtension == ".jpg" || fileExtension == ".png") {
				File file = new File("/upload/" + fileName + fileExtension);
				FileOutputStream fileOutputStream = new FileOutputStream(file);
				fileOutputStream.write(decodedBytes);
				fileOutputStream.close();
			} else {
				throw new ServiceException("jpg만 됨 ㅇㅇ;");
			}
		} catch (Exception e) {
			log.error("에러입니다: {}", e.getMessage());
		}

		insertDto.setBoardPhoto(fileName);
		boardMapper.insert(insertDto);
	}
```