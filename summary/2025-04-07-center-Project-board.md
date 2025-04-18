## <span style="color:#802548">_board 조회수 update시에는 updateTime 안 바뀌게 설정_</span>

- dynamicInsert가 붙은 이유는 조회수 때문이다.
- 조회수를 수정하기 위해 update를 날리는 경우에는 실제 board의 update가 아니다.
- 따라서 조회수 update는 updateDateTime을 바꿔선 안된다.

```java
@DynamicInsert
@Entity
@Table(name = "board")
@AllArgsConstructor
@NoArgsConstructor
@Getter
@Setter
@Builder
@ToString
public class BoardEntity {
	
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "board_seq")
    private Long boardSeq;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_seq", referencedColumnName = "user_seq")
    private UserEntity userEntity;

    @Column(name = "board_title", length = 200, columnDefinition = "varchar(200) default 'Untitled'")
    private String boardTitle;

    @Column(name = "board_content", length = 4000)
    private String boardContent;

    @Column(name = "hit_count")
    private Integer hitCount;

    @Column(name = "create_date", updatable = false)
    @CurrentTimestamp
    private LocalDateTime createDate;

    @Column(name = "update_date")
    @UpdateTimestamp
    private LocalDateTime updateDate;

    @Column(name = "is_deleted")
    private Boolean isDeleted;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "recipe_seq",referencedColumnName = "recipe_seq")
    private RecipeEntity recipeEntity;

    @OneToOne(mappedBy = "boardEntity")
    private BoardImageEntity boardImageEntity;

    public static BoardEntity toEntity(BoardDTO boardDTO, UserEntity userEntity, RecipeEntity recipeEntity) {
        return BoardEntity.builder()
                .boardSeq(boardDTO.getBoardSeq() != null ? boardDTO.getBoardSeq() : null)
                .userEntity(userEntity)
                .boardTitle(boardDTO.getBoardTitle())
                .boardContent(boardDTO.getBoardContent())
                .hitCount(boardDTO.getHitCount())
                .createDate(boardDTO.getCreateDate())
                .updateDate(boardDTO.getUpdateDate())
                .isDeleted(boardDTO.getIsDeleted())
                .recipeEntity(recipeEntity)
                .build();
    }
}
```

- 실제로 아래에 @Modifying을 붙여주면 조회수를 바꿀 때도 updateTime이 갱신되지 않게 된다.
- 아래 쿼리는 concurrency도 보장된다. db 단에서 atomic 연산을 진행하기 때문이다.
- JPA 단에서 entity에서 ++로 dirty checking으로 바꾸는 경우에는 atomic 연산이 일어나지 않는다.
- 자세한 것은 사이트 참조. https://velog.io/@hyeok_1212/%EC%A1%B0%ED%9A%8C%EC%88%98-%EC%A0%95%ED%95%A9%EC%84%B1

```java
public interface BoardRepository extends JpaRepository<BoardEntity, Long> {

    // 조회수(hitCount)만 증가시키는 update 쿼리 (update_date에는 영향을 주지 않음)
    @Modifying
    @Query("update BoardEntity b set b.hitCount = b.hitCount + 1 where b.boardSeq = :boardSeq")
    void incrementHitCount(@Param("boardSeq") Long boardSeq);
}
```

- 혹시 update를 하게 된 이후 바로 해당 값을 사용하게 될 수 있으니, @Modifying에 아래 옵션을 달아두자.
- @Modifying을 달면 entity를 1차 cache에서 바꾸는 기능이 사라진다. 따라서 그냥 1차 캐시를 비워서 새로 조회해오게끔 변경하는 것이다.
- 그래야 update된 내용을 반영하여 제대로 작동할 수 있다.

```java
public interface BoardRepository extends JpaRepository<BoardEntity, Long> {

    // 조회수(hitCount)만 증가시키는 update 쿼리 (update_date에는 영향을 주지 않음)
    @Modifying(clearAutomatically = true)
    @Query("update BoardEntity b set b.hitCount = b.hitCount + 1 where b.boardSeq = :boardSeq")
    void incrementHitCount(@Param("boardSeq") Long boardSeq);
}
```

## <span style="color:#802548">_board 최신 10개 가져올 때는 11개를 가져와서 filter하기_</span>

- 최신 10개를 가져올 때 자기가 포함되어 있을 가능성도 있다.
- 그렇기 때문에 11개를 가져와서 조건에 맞게 filter를 걸고 10개로 limit을 걸어주면 된다.

```java
public List<BoardDTO> selectRecentPostsByUserByTen(Long userSeq, Long currentBoardSeq) {
    UserEntity user = userRepository.findById(userSeq)
            .orElseThrow(() -> new RuntimeException("User not found"));

    List<BoardEntity> boardEntities = boardRepository.findTop11ByUserEntityOrderByCreateDateDesc(user)
                                            .stream()
                                            .filter(board -> (board.getIsDeleted() == null || !board.getIsDeleted())
                                                    && !board.getBoardSeq().equals(currentBoardSeq))
                                            .limit(10)
                                            .collect(Collectors.toList());

    return boardEntities.stream()
            .map(BoardDTO::toDTO)
            .collect(Collectors.toList());
}
```

## <span style="color:#802548">_board 인기 포스트, 최신 포스트 가져오기_</span>


//FIXME. 두 개를 sql로 union 등으로 붙이고, flag column 값을 db 상으로 제조해서 가져오면 어떨까? 농좆에서 했던 것처럼..


```java
public List<BoardDTO> selectPopularPosts() {
    List<BoardEntity> allBoards = boardRepository.findAll();
    List<BoardDTO> popularPosts = allBoards.stream()
        .filter(board -> board.getIsDeleted() == null || !board.getIsDeleted())
        .sorted((b1, b2) -> {
            int heartCount1 = boardHeartRepository.countByBoardAndIsHeartedTrue(b1);
            int heartCount2 = boardHeartRepository.countByBoardAndIsHeartedTrue(b2);
            return Integer.compare(heartCount2, heartCount1);
        })
        .limit(5)
        .map(BoardDTO::toDTO)
        .collect(Collectors.toList());
    return popularPosts;
}

public List<BoardDTO> selectRecentPostsByUserByTen(Long userSeq, Long currentBoardSeq) {
    UserEntity user = userRepository.findById(userSeq)
            .orElseThrow(() -> new RuntimeException("User not found"));

    List<BoardEntity> boardEntities = boardRepository.findTop11ByUserEntityOrderByCreateDateDesc(user)
                                            .stream()
                                            .filter(board -> (board.getIsDeleted() == null || !board.getIsDeleted())
                                                    && !board.getBoardSeq().equals(currentBoardSeq))
                                            .limit(10)
                                            .collect(Collectors.toList());

    return boardEntities.stream()
            .map(BoardDTO::toDTO)
            .collect(Collectors.toList());
}
```

## <span style="color:#802548">_board write시 recipe 목록 가져오기_</span>

- boardWrite 시에는 recipe 목록을 가져온다.
- 필요한 field만 가져와서 트래픽을 최소화한다.

```java
public interface RecipeProjection {
    Long getId();
    String getTitle();
}

public interface RecipeRepository extends JpaRepository<RecipeEntity, Long> {

    @Query("""
             SELECT r.recipeSeq AS id, roc.recipeTitle AS title FROM RecipeEntity r
             JOIN r.recipeOutputEntity roc
             WHERE r.userEntity.userSeq = :userSeq
             AND NOT EXISTS (SELECT 1 FROM r.boardEntity b where b.isDeleted = false)
             AND r.isDeleted = false
             ORDER BY r.recipeSeq DESC
            """)
    List<RecipeProjection> findRecipesByUser(@Param("userSeq") Long userSeq);

}
```



## <span style="color:#802548">_board entity에 응집성 있게 DDD처럼 관리_</span>
- 똑같은 의미의 조건이라면, service에 if문을 만들어서 쓰는 것보다 entity 혹은 DTO에서 직접 method를 관리하는 게 좋다.
- 아래처럼 if문으로 해버리면 바꿔야 할 때 다른 곳에서도 저러한 if문이 있나 전부 찾아서 일일이 수정해야한다.

```java
if (!boardDTO.getThumbnailUrl() != null && !boardDTO.getThumbnailUrl().isEmpty()) {
    return;
}

String thumbnailUrl = boardDTO.getThumbnailUrl();
String savedFileName = "";
if (thumbnailUrl.startsWith("/uploads/")) {
    savedFileName = thumbnailUrl.substring(9);
}

```


- 응집성있게 딱 그것만 고치면 전부 적용되게 class에 method로 만들어준다.

```java
@Getter
@Setter
@Builder
@AllArgsConstructor
@NoArgsConstructor
@ToString
public class BoardDTO {
    .
    .
    .
    
    public boolean isThumbnailUrlEmpty() {
        return StringUtils.hasText(this.thumbnailUrl);
    }

    public boolean isUploadedInCurrentBoard() {
        return this.thumbnailUrl.startsWith("/uploads/");
    }
}
```


## <span style="color:#802548">_board reply_</span>

- 처음에는 아래와 같이 만들었었다.

```java
 // 부모 댓글 (답글이 아닌 댓글) 목록 조회 (삭제되지 않은 댓글만 가져옴)
List<ReplyEntity> parentReplies = replyRepository.findByBoardAndParentReplyIsNullAndIsDeletedFalse(
        boardOpt.get(), Sort.by(Sort.Direction.ASC, "createDate"));

List<ReplyDTO> allReplies = new ArrayList<>();

// 부모 댓글을 순회하며 답글 포함하여 리스트 생성
for (ReplyEntity parent : parentReplies) {
    ReplyDTO parentDTO = ReplyDTO.toDTO(parent);
    allReplies.add(parentDTO); // 부모 댓글 추가

    // 부모 댓글의 자식 댓글(답글) 가져오기
    List<ReplyDTO> childReplies = replyRepository.findByParentReplyAndIsDeletedFalse(parent)
            .stream().map(ReplyDTO::toDTO).collect(Collectors.toList());

    allReplies.addAll(childReplies); // 답글 추가
}

// 페이지네이션 적용 (20개씩)
int start = page * 20;
int end = Math.min(start + 20, allReplies.size());

if (start > allReplies.size()) {
    return Page.empty();
}

List<ReplyDTO> pagedReplies = allReplies.subList(start, end);

return new PageImpl<>(pagedReplies, PageRequest.of(page, 20), allReplies.size());
```

- 그러나 select가 너무 많이 실행됐다. 그래서 아래와 같이 바꾸게 됐다.
- 댓글과 대댓글을 부모댓글- 대댓글 순서대로 가져오게끔 하기 위해서는 COALESCE 함수를 써야만 한다.

```java
@Query("""
    SELECT DISTINCT r FROM ReplyEntity r 
    LEFT JOIN FETCH r.childReplies c
    WHERE r.board = :board 
    AND (r.parentReply IS NULL OR r.parentReply IN 
    (SELECT p FROM ReplyEntity p WHERE p.board = :board AND p.parentReply IS NULL))
    ORDER BY COALESCE(r.parentReply.replySeq, r.replySeq)
""")
Page<ReplyEntity> findRepliesByBoard(@Param("board") BoardEntity board, Pageable pageable);


public Page<ReplyDTO> getReplies(Long boardSeq, int page) {
    Optional<BoardEntity> boardOpt = boardRepository.findById(boardSeq);
    if (boardOpt.isEmpty()) {
        throw new IllegalArgumentException("게시글이 존재하지 않습니다.");
    }

    Pageable pageable = PageRequest.of(page, 10, Sort.by(Sort.Direction.ASC, "createDate"));

    // Fetch paginated parent and child replies in a single query
    Page<ReplyEntity> replyPage = replyRepository.findRepliesByBoard(boardOpt.get(), pageable);

    // Convert to DTOs
    List<ReplyDTO> replyDTOs = replyPage.getContent().stream()
            .map(ReplyDTO::toDTO)
            .collect(Collectors.toList());

    return new PageImpl<>(replyDTOs, pageable, replyPage.getTotalElements());
}
```

- 가져온 댓글을 바탕으로 로그인 아이디가 같은 지 여부, 삭제 여부, 더보기 여부, 대댓글 여부를 판단한다.
- 판단하여 그에 맞게 UI를 구성한다.

```js
function renderComment(item, loginId, isChild) {
    let indentStyle = item.parentReplySeq ? 'style="margin-left: 30px;"' : '';
    let fullText = escapeHTML(item.replyContent);
    let hasMore = fullText.length > 50; // 50자 이상일 때만 "더보기" 버튼 생성

    let tag = `
        <div class="comment" ${indentStyle} data-reply-seq="${item.replySeq}">
            <div class="comment-header">
                <div class="user-info">${escapeHTML(item.replyWriter)}</div>
                ${
                    loginId === item.userId
                        ? `
                    <div class="comment-buttons">

                    ${item.isDeleted ? '' : `<button class="edit-input-btn" onclick="deleteReply(${item.replySeq})">삭제</button>
                        <button class="edit-cancel-btn" onclick="editReply(${item.replySeq}, '${escapeHTML(
                            item.replyContent
                        )}')">수정</button>` }
                        
                    </div>
                `
                        : ''
                }
            </div>
            <div class="user-text">
                <span class="full-text">${item.isDeleted ? '삭제된 댓글입니다' : fullText}</span>
                ${hasMore && !item.isDeleted ? '<button class="more-btn" onclick="toggleExpand(this)">더보기</button>' : ''}
            </div>
    `;
    if (!isChild && loginId) {
        tag += `<button class="reply-btn" onclick="showReplyForm(${item.replySeq})">댓글</button>`;
    }

    tag += `</div>`;
    tag += `<div id="reply-form-${item.replySeq}" class="reply-form" style="display: none; margin-left: 30px;"></div>`;
    // 로그인한 사용자만 답글 버튼 보이게 설정

    return tag;
}
```


//TODO UI/UX 관련해서 나중에 다시 정리

## <span style="color:#802548">_board write 시 image upload 안하게 변경_</span>

- dropzone js를 사용하면 image를 dropzone에 올리는 순간, upload가 되어버리게끔 구성했었다.
- 실제로 사용하지 않고 저장하지 않을 파일들이 저장되어버리는 일이 발생할 수 있다는 의미다.
- url이 /board/upload로 되어있다. 실제 서버의 url mapping을 탄다는 의미다.
- url이 server에서 200 code를 받으면 success event가 발생한다.
    - success event는 dropzone에 있는 내용이 Quill board로 가는 것이다.
    - reader.readAsDataURL(file);를 안 써주면 어떤 작동도 하지 않는다.

```java
const dropzone = new Dropzone('#dropzone', {
	url: '/board/upload',
	maxFiles: 5,
	maxFilesize: 5,
	acceptedFiles: 'image/*',
	addRemoveLinks: true,
	dictDefaultMessage: '',
	dictRemoveFile: '삭제',
	init: function () {

		this.on('success', function (file, response) {
			console.log('업로드 완료:', response);
			if (response && response.fileUrl) {
				var fileUrl = response.fileUrl;
				const reader = new FileReader();
				reader.onload = function (e) {
					var base64Data = e.target.result;
					insertImageToQuill(file, base64Data, fileUrl);
				};
				reader.readAsDataURL(file);
			} else {
				const reader = new FileReader();
				reader.onload = function (e) {
					var base64Data = e.target.result;
					insertImageToQuill(file, base64Data, "");
				};
				reader.readAsDataURL(file);
			}
		});

		this.on('error', function (file, errorMessage) {
			console.error('업로드 실패:', errorMessage);
		});

		this.on('complete', function (file) {
			file.previewElement.classList.add('dz-complete');
		});
	}
});
```

- 삭제하게 되면 image가 서버에서도 삭제된다.

```js
this.on('removedfile', function (file) {
    console.log('Dropzone에서 파일 삭제됨:', file.name);
    let fileKey = file.name + file.size;
    let fileData = uploadedFiles.get(fileKey);
    if (fileData) {
        let imgToRemove = quill.root.querySelector(`img[src="${fileData.base64}"]`);
        if (imgToRemove) {
            imgToRemove.remove();
        }
        if (fileData.url) {
            fetch('/board/deleteFile', {
                method: 'POST',
                headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
                body: 'fileUrl=' + encodeURIComponent(fileData.url)
            })
                .then(response => response.json())
                .then(data => {
                    if (data.error) {
                        console.error('파일 삭제 실패:', data.error);
                    } else {
                        console.log('파일 삭제 성공');
                    }
                })
                .catch(err => console.error('파일 삭제 중 오류:', err));
        }
    }
    uploadedFiles.delete(fileKey);
    if (this.files.length === 0) {
        this.element.classList.remove('dz-started');
    }
    const currentThumbnailUrl = document.getElementById('thumbnailUrl').value;
    if (fileData && fileData.url === currentThumbnailUrl) {
        document.getElementById('thumbnailUrl').value = "";
    }
});
```

- 하지만 근본적으로 저장을 하지 않을 image를 서버에 upload할 필요가 없다.
- 서버에서 image를 올리지 않으므로 success받을 것도 없다.
    - success에 있는 로직을 addedfile에 옮겨준다. 중복확인도 여기서 해줘야 한다.
    - 중복이라면 해당 파일은 삭제해버린다. 중복의 경우에는 정상 작동이 아니므로 도중에 return시킨다.
    - send를 서버에 하지 않으므로 request는 전부 abort시켜준다. 이는 sending에서 다뤄준다.
- 중복된 것의 경우에는 this keyword를 사용하여 Dropzone 객체는 모두 공유하는 멤버변수로서 사용한다.
    - 여기서 this는 class의 instance이며, this.duplicate는 member 변수로서 duplicate가 추가된 것이다.

```js
const dropzone = new Dropzone('#dropzone', {
    url: '#', // server에 request하지 않게 변경
    autoProcessQueue: false, // server에 request하지 않게 변경
    maxFiles: 5,
    maxFilesize: 5,
    acceptedFiles: 'image/*',
    addRemoveLinks: true,
    dictDefaultMessage: '',
    dictRemoveFile: '삭제',
    init: function () {
        this.on('addedfile', function (file) {
            let reader = new FileReader();
            reader.onload = (event) => {
                let base64 = event.target.result;
                
                // base64로 저장된 값을 확인하여 중복된 이미지  올리기 방지
                for (const [key, value] of uploadedFiles) {
                    if (value.base64 === base64) {
                        Swal.fire({
                            icon: 'error',
                            title: '파일 중복',
                            text: '이미 업로드된 파일입니다.',
                            confirmButtonColor: '#ff7f50',
                            confirmButtonText: '확인'
                        });
                        this.duplicate = true;
                        this.removeFile(file);
                        return;
                    }
                }

                uploadedFiles.set(file.name, { file, base64 });
                insertImageToQuill(file, base64);
            }
            reader.readAsDataURL(file);
            this.element.classList.add('dz-started');
        });
    }
});

this.on("sending", function(file, xhr, formData) {
    //upload 시 server에 request하지 않게 변경
    xhr.abort();
});
```

- 파일이 삭제될 떄도 서버를 들를 이유가 아예없다.
- 혹시 중복된 파일이 삭제되는 것이라면, Quillboard에서 지우면 안된다. 
- 그를 위해서 위에서 중복확인 변수를 추가한 것이다. 
- 중복이라서 지우는 것은 이후 작동이 없어야 하므로 중복 flag만 원복하고 return시킨다.

```js
this.on('removedfile', function (file) {
    let fileKey = file.name;
    let fileData = uploadedFiles.get(fileKey);

    //중복 파일이라면 quill에서 제거하지 않고, 새로 upload된 파일이면 제거
    if (!this.duplicate) {
        removeImageFromQuillBoard(fileData.base64);
    } else {
        this.duplicate = false;
        return;
    }

    //dropzone에 image가 사라졌으므로 image가 없음을 알려주기 위해 css 제거
    if (this.files.length === 0) {
        this.element.classList.remove('dz-started');
    }
    uploadedFiles.delete(fileKey);
    preventRequestIfRemovedImageIsThumbnail(fileData.base64);
});
```

- 이와 더불어 알아보기 어려운 js들도 전부 이름을 준다.

```js
function insertImageToQuill(file, base64Data, fileUrl) {
	let range = quill.getSelection();
	let insertIndex = range ? range.index : quill.getLength();
	quill.insertEmbed(insertIndex, 'image', base64Data);
	quill.setSelection(insertIndex + 1);

	let fileKey = file.name + file.size;
	uploadedFiles.set(fileKey, { file: file, base64: base64Data, url: fileUrl });

	file.previewElement.classList.add('dz-complete');

	let thumbnailLabel = document.createElement('div');
	thumbnailLabel.classList.add('thumbnail-label');
	thumbnailLabel.textContent = '썸네일로 지정';
	thumbnailLabel.style.display = 'none';
	file.previewElement.insertBefore(thumbnailLabel, file.previewElement.firstChild);

	file.previewElement.addEventListener('click', function (e) {
		if (e.target.classList.contains('dz-remove')) return;
		document.querySelectorAll('.dz-preview').forEach(function (preview) {
			preview.classList.remove('thumbnail-selected');
			let label = preview.querySelector('.thumbnail-label');
			if (label) { label.style.display = 'none'; }
		});
		file.previewElement.classList.add('thumbnail-selected');
		thumbnailLabel.style.display = 'block';
		document.getElementById('thumbnail').value = base64Data;
		document.getElementById('thumbnailUrl').value = fileUrl;
		console.log('썸네일 지정:', fileUrl);
	});
}
```

- 우선 단위별로 구분하여 주석으로 문맥을 만들어준다. 


```js
function insertImageToQuill(file, base64Data, fileUrl) {
	let range = quill.getSelection();
	let insertIndex = range ? range.index : quill.getLength();
	quill.insertEmbed(insertIndex, 'image', base64Data);
	quill.setSelection(insertIndex + 1);

	let fileKey = file.name + file.size;
	uploadedFiles.set(fileKey, { file: file, base64: base64Data, url: fileUrl });

	file.previewElement.classList.add('dz-complete');

    //thunmail 생성
	let thumbnailLabel = document.createElement('div');
	thumbnailLabel.classList.add('thumbnail-label');
	thumbnailLabel.textContent = '썸네일로 지정';
	thumbnailLabel.style.display = 'none';
	file.previewElement.insertBefore(thumbnailLabel, file.previewElement.firstChild);

    //image click 시 thumnail 변경하는 listner
	file.previewElement.addEventListener('click', function (e) {
		if (e.target.classList.contains('dz-remove')) return;

        //thumnail 변경
		document.querySelectorAll('.dz-preview').forEach(function (preview) {
			preview.classList.remove('thumbnail-selected');
			let label = preview.querySelector('.thumbnail-label');
			if (label) { label.style.display = 'none'; }
		});
		file.previewElement.classList.add('thumbnail-selected');
		thumbnailLabel.style.display = 'block';
		document.getElementById('thumbnail').value = base64Data;
		document.getElementById('thumbnailUrl').value = fileUrl;
		console.log('썸네일 지정:', fileUrl);
	});
}
```

- stage를 주석으로 나눠서 달아줘도 되지만 함수를 달아주는 게 더 유지보수하기 편하다.

```js
function insertImageToQuill(file, base64) {
    let range = quill.getSelection();
    let insertIndex = range ? range.index : quill.getLength();
    quill.insertEmbed(insertIndex, 'image', base64);
    quill.setSelection(insertIndex + 1);

    let fileKey = file.name;
    uploadedFiles.set(fileKey, { file, base64 });

    file.previewElement.classList.add('dz-complete');

    const thumbnailLabel = generateThumbnailLabel(file);

    file.previewElement.addEventListener('click', function (e) {
        if (e.target.classList.contains('dz-remove')) {
            return;
        }
        updateThumbnailSelection(file, thumbnailLabel, base64);
    });
}
```


## <span style="color:#802548">_board update 시 image 받아와서 dropzone에 preload하게 변경_</span>

- page가 load 되자마자 preload된 이미지를 보여주기 위해서는 dropzone.js의 emit을 활용해야 한다.
- dropzone의 영역에 넣을 image를 Quill.js에서 가져온다.
- update에서도 server upload를 하지 않으므로 #으로 url을 설정한다.

```js
const dropzone = new Dropzone('#dropzone', {
    url: '#', // upload 시 server에 request하지 않게 변경
    autoProcessQueue: false, // upload 시 server에 request하지 않게 변경
    maxFiles: 5,
    maxFilesize: 5,
    acceptedFiles: 'image/*',
    addRemoveLinks: true,
    dictDefaultMessage: '',
    dictRemoveFile: '삭제',
    init: function () {
        let dropzoneInstance = this;

        // load 시에 thymeleaf로 그린 HTML에서 preload에 담을 image를 가져옴. load 시에 한번만 작동
        const editor = document.querySelector('.ql-editor');
        const imgElements = editor.querySelectorAll('img');
        const imgSrcs = Array.from(imgElements).map(img => img.getAttribute('src') || '');
        imgSrcs.forEach((base64, index) => {
            // 저장한 image를 dropzone에 preload를 적용 
            let mockFile = {
                name: "uploaded-image-" + index, 
                size: calculateImageSize(base64), 
                type: 'image/*'
            };
            dropzoneInstance.emit("addedfile", mockFile);
            dropzoneInstance.emit("thumbnail", mockFile, base64);
            dropzoneInstance.files.push(mockFile);
            mockFile.previewElement.classList.add('dz-complete'); 

            // thumbnail인 이미지는 thumnail 표시 CSS 적용
            let thumbnailLabel = generateThumbnailLabel(mockFile);
            setThumbnailOnLoad(mockFile, thumbnailLabel, base64);

            // dropzone click 시 thumbnailImage 변경되게 적용
            handleClickDropzoneImage(mockFile, thumbnailLabel, base64);


            // dropzone image에서 제거 버튼 누르면 dropzone에서 삭제되게 적용
            let removeButton = mockFile.previewElement.querySelector(".dz-remove");
            handleDropzoneImageRemoveBtn(removeButton, dropzoneInstance, mockFile, base64);

            // Base64 문자열을 디코딩하여 바이너리 데이터로 변환하는 함수
            const file = convertBase64ToFile(base64, mockFile);

            const fileKey = mockFile.name;
            uploadedFiles.set(fileKey, { file, base64  });
        });
    }
})
```

- 순서는 저장된 image의 src, base64를 가져온다.
- 이 중 thumnail인인 이미지에 thunmail css를 씌운다.
- 혹시 thunmail이 변경되면 바꿔주는 처리를 해준다. 
- dropzone에서 삭제 버튼을 누르면 dropzone에서 제거되게 해야하는데, 이전 것의 preload여도, 이번 upload여도 잘 작동하게 한다.
- 성공적으로 넣었다면 이를 객체에 보관한다.

```js
function generateThumbnailLabel(file) {
    let thumbnailLabel = document.createElement('div');
    thumbnailLabel.classList.add('thumbnail-label');
    thumbnailLabel.textContent = '썸네일로 지정';
    thumbnailLabel.style.display = 'none';
    file.previewElement.insertBefore(thumbnailLabel, file.previewElement.firstChild);
    return thumbnailLabel;
}

function setThumbnailOnLoad(mockFile, thumbnailLabel, base64) {
    const dropZoneImgElements = mockFile.previewElement.querySelectorAll('img');
    const dropzoneImgSrcs = Array.from(dropZoneImgElements).map(img => img.getAttribute('src') || '');
    for (let i = 0; i < dropzoneImgSrcs.length; i++) {
        if (dropzoneImgSrcs[i] === document.getElementById('thumbnail').value) {
            mockFile.previewElement.classList.add('thumbnail-selected');
            thumbnailLabel.style.display = 'block';
            document.getElementById('thumbnail').value = base64;
            document.getElementById('thumbnailUrl').value = base64;
        }
    }
}

function handleClickDropzoneImage(mockFile, thumbnailLabel, base64) {
    mockFile.previewElement.addEventListener('click', function (e) {
        //삭제버튼을 누른 경우 차후 진행 방지
        if (e.target.classList.contains('dz-remove')) {
            return;
        }
        updateThumbnailSelection(mockFile, thumbnailLabel, base64);
    });
}

/**
 * 
 * @param {File} file  dropzonefile
 * @param {HTMLElement} thumbnailLabel 
 * @param {base64String} base64 
 */
function updateThumbnailSelection(file, thumbnailLabel, base64) {
    //썸네일 이미지를 바꾸기 위해 이전 썸네일 이미지 표시 css를 초기화
    document.querySelectorAll('.dz-preview').forEach(function (preview) {
        preview.classList.remove('thumbnail-selected');
        let label = preview.querySelector('.thumbnail-label');
        if (label) {
            label.style.display = 'none';
        }
    });
     //새로 선택된 썸네일 이미지를 보여주기 위해 이미지 표시 css 추가
     file.previewElement.classList.add('thumbnail-selected');
     thumbnailLabel.style.display = 'block';
     // 새로 지정한 썸네일이면 hidden input 갱신
    document.getElementById('thumbnail').value = base64;
    document.getElementById('thumbnailUrl').value = base64;
}

function handleDropzoneImageRemoveBtn(removeButton, dropzoneInstance, mockFile, base64) {
    removeButton.addEventListener("click", function (e) {
        e.preventDefault();
        e.stopPropagation();
        dropzoneInstance.removeFile(mockFile);
        removeImageFromQuillBoard(base64);
    });
}

function convertBase64ToFile(base64, mockFile) {
    const base64String = base64.replace(/^data:image\/(png|jpeg);base64,/, '');
    const binaryData = base64ToBinary(base64String);
    const blob = new Blob([binaryData], { type: 'image/*' });
    const file = new File([blob], mockFile.name, { type: 'image/*' });
    return file;
}

function removeImageFromQuillBoard(base64) {
    let imgToRemove = quill.root.querySelector(`img[src="${base64}"]`);
    if (imgToRemove) {
        imgToRemove.remove();
    }
}
```


- 만약 file 객체가 아닌, 특수 file객체라면 아래처럼 jsdoc에 명시해준다.
- 그렇지 않으면 오류가 나는 경우가 생길 수 있다. 

```js
/**
 * Inserts an image into Quill.
 *
 * @param {File} file - dropzone file.
 * @param {string} base64 - The base64-encoded image data.
 */
function insertImageToQuill(file, base64) {
    let range = quill.getSelection();
    let insertIndex = range ? range.index : quill.getLength();
    quill.insertEmbed(insertIndex, 'image', base64);
    quill.setSelection(insertIndex + 1);

    let fileKey = file.name;
    uploadedFiles.set(fileKey, { file, base64 });

    file.previewElement.classList.add('dz-complete');

    const thumbnailLabel = generateThumbnailLabel(file);

    file.previewElement.addEventListener('click', function (e) {
        if (e.target.classList.contains('dz-remove')) {
            return;
        }
        updateThumbnailSelection(file, thumbnailLabel, base64);
    });
}
```

- 가령 아래의 dropzone.removeFile에 들어가는 parameter는 그냥 file 객체가 아니라 dropzone의 file 객체여야 한다.
- 따라서 uploadedFile에 넣은 그냥 file 객체를 사용할 수 없다. 
- boardWrite에서는 모두 dropzone file이었다. 하지만 preload로 인해 boardUpdate.js에서는 그냥 file 객체도 있다.
    - 따라서 dropzone.files에서 find를 하여 dropzone의 파일 객체를 가져오는 작업이 필요하다.


```js
quill.on('text-change', function () {
    let quillImages = new Set([...quill.root.querySelectorAll('img')].map(img => img.src));
    for (let [fileKey, fileData] of uploadedFiles.entries()) {
        if (!quillImages.has(fileData.base64)) {
            //base64 -> file 객체화한 file과 dropzone자체에서 만든 file이 서로 다른 객체라서 name만 같게 하여 dropzone 객체의 file을 찾아오게 함.
            const fileToRemove = dropzone.files.find(file => file.name === fileData.file.name);
            if (fileToRemove) {
                dropzone.removeFile(fileToRemove);
            } 
            uploadedFiles.delete(fileKey);
            preventRequestIfRemovedImageIsThumbnail(fileData?.base64);
        }
    }
});
```


## <span style="color:#802548">_board image upload base64가 아닌 file path로 변경_</span>
//TODO


## <span style="color:#802548">_board image upload compress하게 변경_</span>
//TODO


## <span style="color:#802548">_Reply에서 Distinct JPQL 제거_</span>

- Distinct를 지우기 위해 Set을 활용하려면 EqualsAndHashCode를 재정의 해야한다.
- replySeq를 기준으로 중복을 제거해주면 되기 때문에 아래처럼 써준다.

```java
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Entity
@Table(name = "reply")
@EqualsAndHashCode(of = "replySeq")
public class ReplyEntity {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "reply_seq")
    private Long replySeq;

.
.
}
```

- 참고로 pageable을 넣어도 JPA에서 Page를 return해야만 Limit, offset이 붙는다. List를 return하면 안 된다.
- DISTINCT keyword를 제거한다. 대신 Pageable에서 count를 가져올 때 별도의 query를 만들어줘야 한다.
- JPQL에서 DISTINCT를 써야 JPA에서 알아서 중복을 처리해서 Pageable의 getTotalElements에서 paging이 제대로 처리된다.
    - 하지만 그게 안된다면 아래처럼 countQuery를 직접 넣어주자. 
    - 처음부터 countQuery는 넣어주는 게 좋다.

```java
public interface ReplyRepository extends JpaRepository<ReplyEntity, Long>{

    @Query(value = """
        SELECT r FROM ReplyEntity r 
        LEFT JOIN FETCH r.childReplies c
        WHERE r.board = :board 
        AND (r.parentReply IS NULL OR r.parentReply IN 
        (SELECT p FROM ReplyEntity p WHERE p.board = :board AND p.parentReply IS NULL))
        ORDER BY COALESCE(r.parentReply.replySeq, r.replySeq)
    """, countQuery = """
        select count(r) FROM ReplyEntity r
         WHERE r.board = :board 
""")
    Page<ReplyEntity> findRepliesByBoard(@Param("board") BoardEntity board, Pageable pageable);
}
```

- JPQL에서 distinct를 지웠으니 Java에서 중복 객체를 제거해줘야 한다.
- 페이징이라 순서도 중요하기 때문에 LinkedHashSet을 사용해준다.

```java
 public Page<ReplyDTO> getReplies(Long boardSeq, int page) {
    Optional<BoardEntity> boardOpt = boardRepository.findById(boardSeq);
    if (boardOpt.isEmpty()) {
        throw new IllegalArgumentException("게시글이 존재하지 않습니다.");
    }

    Pageable pageable = PageRequest.of(page, 10, Sort.by(Sort.Direction.ASC, "createDate"));

    // Fetch paginated parent and child replies in a single query
    Page<ReplyEntity> replyPage = replyRepository.findRepliesByBoard(boardOpt.get(), pageable);
    List<ReplyEntity> deduplicated = new ArrayList<>(new LinkedHashSet<>(replyPage.getContent()));
    
    // Convert to DTOs
    List<ReplyDTO> replyDTOs = deduplicated.stream()
                                        .map(ReplyDTO::toDTO)
                                        .collect(Collectors.toList());

    return new PageImpl<>(replyDTOs, pageable, replyPage.getTotalElements());
    }
```


- LinkedHashSet class는 EqualsAndHasChode를 구현해야하므로 Entity class를 건드려야 한다.
- 또한 해당 정렬 기준이 Entity에 명확하게 지정되기 떄문에 Enttiy를 이용하는 서비스에서 기준을 바꾸기 어렵다.
- 그러나 Map을 이용하게 되면 LinkedhashMap을 만들면서 기준을 유연하게 바꿀 수 있다.
- 따라서 아래처럼 Map을 사용하는 게 더 유연한 이용이 가능하다. 각 서비스별로 필요할 때마다 바꿀 수 있기 때문이다.

```java
public Page<ReplyDTO> getReplies(Long boardSeq, int page) {
    Optional<BoardEntity> boardOpt = boardRepository.findById(boardSeq);
    if (boardOpt.isEmpty()) {
        throw new IllegalArgumentException("게시글이 존재하지 않습니다.");
    }

    Pageable pageable = PageRequest.of(page, 10, Sort.by(Sort.Direction.ASC, "createDate"));

    // Fetch paginated parent and child replies in a single query
    Page<ReplyEntity> replyPage = replyRepository.findRepliesByBoard(boardOpt.get(), pageable);

    List<ReplyEntity> deduplicatedList = replyPage.getContent().stream()
                                            .collect(Collectors.toMap(
                                                ReplyEntity::getReplySeq,
                                                Function.identity(),
                                                (a, b) -> a,
                                                LinkedHashMap::new
                                            ))
                                            .values()
                                            .stream()
                                            .toList();
    
    // Convert to DTOs
    List<ReplyDTO> replyDTOs = deduplicatedList.stream()
                                        .map(ReplyDTO::toDTO)
                                        .collect(Collectors.toList());

    return new PageImpl<>(replyDTOs, pageable, replyPage.getTotalElements());
}
```