## <span style="color:#802548">_GITガイダンス作成_</span>

- GITを使うことが初めてだった人が多かったため、GITの使用説明書を作成しました。
- ステージングエリア、ローカルリポジトリ、リモートリポジトリ、ブランチなどの基本的な概念からGITコマンドまで必要なことを教わりました。
- それを見ながら練習できるようにさせるため、シナリオも作りました。

```
リモートリポジトリローカルにコピーする：git clone

ファイル作成: untracked
Gitリポジトリの状態を確認する: git status
ステージングに追加する: git add 
ローカルリポジトリに変更を確定する: git commit
コミット履歴を確認: git log
確定した変更をリモートリポジトリに送信: git push

ブランチを作る: git switch -C [branch name]
ファイル修正: changed
ステージングに追加する: git add 
ローカルリポジトリに変更を確定する: git commit（コミット）
確定した変更をリモートリポジトリに送信: git push（プッシュ）

ブランチお互いに融合する: pull request(プルリクエスト)

リモートリポジトリの変更をローカルに同期する: git pull
ファイル修正: changed
ファイルの変更を元に戻す: git restore
ファイル修正: changed
途中の作業を一時的に保存する: git stash
```

## <span style="color:#802548">_GIT戦略_</span>
- GITの初心者がいるため、管理戦略に関しては簡単な戦略、トランクベース開発戦略にしました。
    -DEVブランチを設けず、FEATUREブランチから直接MAINブランチにマージする戦略を採用していました。
    - 複雑性を下げて実務を体験することができると思いました。
- チーム全体で、こまめにプルリクエストを出すよう心がけました。
    - ブランチの統合が遅れると、Gitのコンフリクトが増え、解決が困難になるためです。
    - それと朝着いた直後、昼ご飯食べてIT授業が始まったばかりで、授業が全部終わる30分前最、つまり小限3回はプルリクエストするようにしました。
    - その結果、大きなコンフリクトが発生しなかったと思います。


## <span style="color:#802548">_JavaScriptクロージャの使用例①_</span>

- 以前の派遣プロジェクトでクロージャを作りましたが、難しいと拒否されたことがあります。
- 今回はグローバル変数を使わないでクロージャを使おうと思いました。
- グローバル変数を使用すると、開発者ツールで値を改ざんされ、バリデーションロジックを迂回してサーバーに送信される恐れがありました。
- 一方、ローカル変数であればそのような心配がないため、より安全性の高い実装が可能になります。
- 多重クリックができないようにするため、サーバーからレスポンスを受け取ってからクリックできるようにしました。
    - そのためFINALLY文でisClicked変数を初期化します。
    - 多重クリックが発生すると、同じデータが重複して登録され、データの整合性が損なわれる恐れがあります。

```js
function createToggleBookmark() {
    let isClicked = false; 

    return async function toggleBookmark() {

        if (isClicked) {
           .
           .
           .
        }


        isClicked = true; 

        const data = {
            title,
            recipeCondition: { usage, menu, taste, level },
            outputContent,
            nonce
        };

        try {
            const response = await fetch('/recipe/history/save', {
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(data),
                method: 'POST'
            });
        .
        .
        .

        } catch (error) {
            console.error("Error:", error);
        } finally {
            isClicked = false;
        }
    };
}
```

- サーバーでデータの登録が完了したら、同じデータを重複して登録してはいけません。
- 登録が成功した場合、そのデータの主キー（ID）を返します。
- オートインクリメントを使用しているため、最初に登録されたデータの主キーは 1 から始まり、0 ではありません。
- そのため、キャプチャ変数には初期値として 0 を設定します。登録が完了するとその値は 0 ではなくなるため、0 でない場合はすでに登録されていると判断できます。


## <span style="color:#802548">_JavaScriptクロージャの使用例②_</span>


```js
function createToggleBookmark() {
    let isClicked = false; 
    let recipeSeq = 0; 

    return async function toggleBookmark() {

        if (isClicked) {
            alert("もうクリックしました")
            return;
        }

        if (recipeSeq !== 0) {
            alert("もう保存されたデータです")
            return;
        }


        isClicked = true; 

        const data = {
            title,
            recipeCondition: { usage, menu, taste, level },
            outputContent,
            nonce
        };

        try {
            const response = await fetch('/recipe/history/save', {
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(data),
                method: 'POST'
            });
        .
        .
        .

        recipeSeq = await response.json();

        } catch (error) {
            console.error("Error:", error);
        } finally {
            isClicked = false;
        }
    };
}

//実行コンテキストを形成する
const toggleBookmark = createToggleBookmark();
document.querySelector('.bookmark-btn').addEventListener('click', toggleBookmark);
```


## <span style="color:#802548">_JavaScriptクロージャの使用例③_</span>

- 他の関数でキャプチャされた変数も、getterのような関数を通じて別の関数から参照できます。

```js
const createPasswordChecker = () => {
    let isPasswordCorrect = false;

    async function checkPw() {
        try {
            const response = await fetch("/mypage/checkPassword", {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify({ currentPassword: password.value })
            });
            const data = await response.json();
            isPasswordCorrect = data;

            validateForm();

            return isPasswordCorrect;
        } catch (error) {
            console.error("Error:", error);
            return false;
        }
    }

    const debouncedCheckPw = debounce(checkPw, 300);

    function isPasswordValid() {
        return isPasswordCorrect;
    }

    return { checkPw: debouncedCheckPw, isPasswordValid };
};

password.addEventListener("keyup", async () => {
    await passwordChecker.checkPw();
});
```

- 実行コンテキストを作ってからgetterのような関数を通じて参照した例です。

```js
function validateForm() {
    const isNickNameValid = nickName.value.trim().length >= 2 && nickName.value.trim().length <= 11;
    const isNewPwValid = newPW.value.length >= 8 && newPW.value === newPWCheck.value;

    const nickNameLengthMessage = document.getElementById("nickNameLengthMessage");
    nickNameLengthMessage.style.display = isNickNameValid ? "none" : "block";

    if (newPasswordSection.style.display === "none") {
        changeInfoBtn.disabled = !(passwordChecker.isPasswordValid() && isNickNameValid);
    } else {
        changeInfoBtn.disabled = !(isNickNameValid && isNewPwValid);
    }

    changePwBtn.disabled = !(passwordChecker.isPasswordValid() && isNickNameValid);
}

const passwordChecker = createPasswordChecker();
```


## <span style="color:#802548">_最適化：凝集性_</span>

- サービスクラスにif文を作るよりentityやDTOで直接メソッドを管理した方が良いです。
- サービスクラスでif文で処理すると、そのif文が複数のクラスで使われた場合、変更する時、他のところにもそのようなif文があるか全部探していちいち修正する必要があります。

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


- 凝集性があるように、つまり、一つのクラスだけを修正すれば他のクラスでも全部適用されるようにクラスのメソッドにします。

```java
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

- クラスに入れたメソッドなら一括で適用されるので運用保守が便利になります。
- BoardInsertServiceクラスやBoardUpdateServiceクラスを修正しなくて一つのBoardDTOだけ修正することで十分です。

```java
public class BoardInsertService {

    public void insertBoard(BoardDTO boardDTO) {
        .
        .
        if (boardDTO.isThumbnailUrlEmpty()) {
            return;
        }

        .
        .
        String thumbnailUrl = boardDTO.getThumbnailUrl();
        String savedFileName = "";
        if (boardDTO.isUploadedInCurrentBoard()) {
            savedFileName = thumbnailUrl.substring(9);
        }
        
        .
        .

    }
    
}

public class BoardUpdateService {

    public void updateBoard(BoardDTO boardDTO) {
        .
        .
        if (boardDTO.isThumbnailUrlEmpty()) {
            return;
        }

        .
        .
        String thumbnailUrl = boardDTO.getThumbnailUrl();
        String savedFileName = "";
        if (boardDTO.isUploadedInCurrentBoard()) {
            savedFileName = thumbnailUrl.substring(9);
        }

        .
        .
    }
}
```



## <span style="color:#802548">_最適化：可読性向上_</span>

- 最初はコードの作成に集中したので読み取りにくい状態でした。

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
	thumbnailLabel.textContent = 'サムネイルとして指定する';
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
		console.log('サムネイルとして指定する:', fileUrl);
	});
}
```

- 可読性を向上するため、段階を区切って段階ごとに注を付けて分かりやすくしました。


```js
function insertImageToQuill(file, base64Data, fileUrl) {
	let range = quill.getSelection();
	let insertIndex = range ? range.index : quill.getLength();
	quill.insertEmbed(insertIndex, 'image', base64Data);
	quill.setSelection(insertIndex + 1);

	let fileKey = file.name + file.size;
	uploadedFiles.set(fileKey, { file: file, base64: base64Data, url: fileUrl });

	file.previewElement.classList.add('dz-complete');

    //サムネイル表示生成
	let thumbnailLabel = document.createElement('div');
	thumbnailLabel.classList.add('thumbnail-label');
	thumbnailLabel.textContent = 'サムネイルとして指定する';
	thumbnailLabel.style.display = 'none';
	file.previewElement.insertBefore(thumbnailLabel, file.previewElement.firstChild);

    //画像クリック時にサムネイルを更新するリスナー
	file.previewElement.addEventListener('click', function (e) {
		if (e.target.classList.contains('dz-remove')) return;

        //サムネイル変更
		document.querySelectorAll('.dz-preview').forEach(function (preview) {
			preview.classList.remove('thumbnail-selected');
			let label = preview.querySelector('.thumbnail-label');
			if (label) { label.style.display = 'none'; }
		});
		file.previewElement.classList.add('thumbnail-selected');
		thumbnailLabel.style.display = 'block';
		document.getElementById('thumbnail').value = base64Data;
		document.getElementById('thumbnailUrl').value = fileUrl;
		console.log('サムネイルとして指定する:', fileUrl);
	});
}
```

- 各段階を関数化し、関数名で処理の内容が分かるようにします。
- コメントは更新されない場合があるからも関数化の理由の一つです。

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

- JavaScriptのファイルクラスではなく、Dropzoneクラスのファイルインスタンスが必要なです。
- それを明確に表示するため、JSDOCを使用します。

```js
/**
 * 
 * @param {File} file  Dropzoneのインスタンス。JavaScriptのFileとは別物
 * @param {HTMLElement} thumbnailLabel 
 * @param {base64String} base64 
 */
function updateThumbnailSelection(file, thumbnailLabel, base64) {

    //　サムネイル画像を切り替えるために、以前のサムネイル表示用のCSSをリセットします。
    document.querySelectorAll('.dz-preview').forEach(function (preview) {
        preview.classList.remove('thumbnail-selected');
        let label = preview.querySelector('.thumbnail-label');
        if (label) {
            label.style.display = 'none';
        }
    });

     //　新しく選択されたサムネイル画像を表示するために、対応するCSSを追加します。
     file.previewElement.classList.add('thumbnail-selected');
     thumbnailLabel.style.display = 'block';

     //　新しく指定されたサムネイル画像に対応するように、サーバーに送信するデータを更新します。　
    document.getElementById('thumbnail').value = base64;
    document.getElementById('thumbnailUrl').value = base64;
}
```


## <span style="color:#802548">_最適化：並行性制御_</span>

- 조회 수가 동시에 증가되지 않게끔 하는 로직이 필요했다.


```java
public interface BoardRepository extends JpaRepository<BoardEntity, Long> {

    // 조회수(hitCount)만 증가시키는 update 쿼리 (update_date에는 영향을 주지 않음)
    @Modifying
    @Query("update BoardEntity b set b.hitCount = b.hitCount + 1 where b.boardSeq = :boardSeq")
    void incrementHitCount(@Param("boardSeq") Long boardSeq);
}
```

## <span style="color:#802548">_最適化：安全性_</span>
- リプレイ攻撃やリロードによる無限保存を防ぐために、nonce値を導入しました。
- まず、UUIDで生成した nonce をセッションに保存します。

```java
@PostMapping("/recipe/chatGPT")
@ResponseBody
public void viewRecipeOutput(@RequestBody RecipeUserRequestDTO recipeUserRequestDTO, HttpSession session) {

    RecipeUserResponseDTO response = recipeService.getRecipeResponse(recipeUserRequestDTO);
    
    session.setAttribute("recipe", response);
    String newUUID = UUID.randomUUID().toString(); 
    session.setAttribute("nonce", newUUID);
}
```

- モデルに格納して送ったnonceは、HTMLでhiddenフィールドとして保存されます。

```html
<input type="hidden" th:value="${nonce}" name="nonce" id="nonce">
<input type="hidden" th:value="${recipe.recipeConditionDTO.usage}" name="usage" id="usage">
<input type="hidden" th:value="${recipe.recipeConditionDTO.menu}" name="menu" id="menu">
<input type="hidden" th:value="${recipe.recipeConditionDTO.taste}" name="taste" id="taste">
<input type="hidden" th:value="${recipe.recipeConditionDTO.level}" name="level" id="level">
```

- 保有しているnonce値をJavaScriptオブジェクトに設定し、サーバーに送信します。

```js
 const title = document.querySelector(".recipe-title").textContent;
const outputContent = document.querySelector(".recipe-info").innerHTML;
const usage = document.querySelector("input[name='usage']").value;
const menu = document.querySelector("input[name='menu']").value;
const taste = document.querySelector("input[name='taste']").value;
const level = document.querySelector("input[name='level']").value;
const nonce = document.querySelector("input[name='nonce']").value;

const data = {
    title,
    recipeCondition: { usage, menu, taste, level },
    outputContent,
    nonce
};
```


- nonce値を確認して保存し、保存が成功した場合はnonce値をセッションから削除して、リプレイやリロードによる無限保存を防ぎます。

```java
@PostMapping("/recipe/history/save")
@ResponseBody
public ResponseEntity<Long> saveRecipeHistory(@RequestBody RecipeHistroyRequsetDTO recipeHistroyRequsetDTO, HttpSession session) throws InterruptedException {
    String userNonce = (String) session.getAttribute("nonce");

    if (!StringUtils.hasText(userNonce) || !recipeHistroyRequsetDTO.getNonce().equals(userNonce)) {
        throw new RuntimeException("nonce値が存在しません。");
    }

    Long savedRecipeSeq = recipeHistoryService.saveRecipeAndReturnSavedPK(recipeHistroyRequsetDTO);
    if(savedRecipeSeq != 0) {
        session.removeAttribute("nonce");   
    }
    
    return ResponseEntity.ok(savedRecipeSeq);
}
```


## <span style="color:#802548">_最適化：可読性のための JPQL_</span>


- 最初は下記のようにクエリメソッドを活用しようと試みましたが、メソッド名が長くなり、可読性が下がると感じました。

```java
public interface RecipeMyPageRepository extends JpaRepository<RecipeEntity, Long> {

    List<RecipeEntity> findByUserEntity_UserSeqAndRecipeInputKeywordEntityListIsNotNullAndRecipeOutputEntityIsNotNull(Long userSeq);
    
} 
```

- 従って、下記のように JPQL を使用する形に変更しました。

```java
public interface RecipeMyPageRepository extends JpaRepository<RecipeEntity, Long>{
    
    @Query("""
        SELECT DISTINCT r FROM RecipeEntity r
        JOIN r.recipeInputKeywordEntityList k
        WHERE r.userEntity.userSeq = :userSeq
        AND r.recipeOutputEntity IS NOT NULL
    """)
    Page<RecipeEntity> findRecipesWithPagination(
        @Param("userSeq") Long userSeq,
        Pageable pageable
    );
    
} 
```

- OneToOneやOneToManyなどのエンティティ間の関連を表すアノテーションによって、複数のエンティティがまとめて取得されるようになります。
- 実際には冗長なクエリが多いため、一括で取得してJava側でパースする必要があるように思われました。

```sql
Hibernate: 
    select
        distinct re1_0.recipe_seq,
        re1_0.created_at,
        re1_0.user_seq 
    from
        recipe re1_0 
    join
        recipe_input_keyword rikel1_0 
            on re1_0.recipe_seq=rikel1_0.recipe_seq 
    left join
        recipe_output_content roe1_0 
            on re1_0.recipe_seq=roe1_0.recipe_seq 
    where
        re1_0.user_seq=? 
        and roe1_0.recipe_output_content_seq is not null 
    order by
        re1_0.created_at 
    limit
        ?

Hibernate: 
    select
        roe1_0.recipe_output_content_seq,
        roe1_0.output_content,
        roe1_0.recipe_seq,
        roe1_0.recipe_title 
    from
        recipe_output_content roe1_0 
    where
        roe1_0.recipe_seq=?

Hibernate: 
    select
        ue1_0.user_seq,
        ue1_0.created_at,
        ue1_0.is_deleted,
        ue1_0.roles,
        ue1_0.updated_at,
        ue1_0.user_email,
        ue1_0.user_id,
        ue1_0.user_name,
        ue1_0.user_password 
    from
        user ue1_0 
    where
        ue1_0.user_seq=?

Hibernate: 
    select
        roe1_0.recipe_output_content_seq,
        roe1_0.output_content,
        roe1_0.recipe_seq,
        roe1_0.recipe_title 
    from
        recipe_output_content roe1_0 
    where
        roe1_0.recipe_seq=?

Hibernate: 
    select
        roe1_0.recipe_output_content_seq,
        roe1_0.output_content,
        roe1_0.recipe_seq,
        roe1_0.recipe_title 
    from
        recipe_output_content roe1_0 
    where
        roe1_0.recipe_seq=?

Hibernate: 
    select
        count(distinct re1_0.recipe_seq) 
    from
        recipe re1_0 
    join
        recipe_input_keyword rikel1_0 
            on re1_0.recipe_seq=rikel1_0.recipe_seq 
    left join
        recipe_output_content roe1_0 
            on re1_0.recipe_seq=roe1_0.recipe_seq 
    where
        re1_0.user_seq=? 
        and roe1_0.recipe_output_content_seq is not null

Hibernate: 
    select
        rikel1_0.recipe_seq,
        rikel1_0.keyword 
    from
        recipe_input_keyword rikel1_0 
    where
        rikel1_0.recipe_seq=?

Hibernate: 
    select
        rikel1_0.recipe_seq,
        rikel1_0.keyword 
    from
        recipe_input_keyword rikel1_0 
    where
        rikel1_0.recipe_seq=?

Hibernate: 
    select
        rikel1_0.recipe_seq,
        rikel1_0.keyword 
    from
        recipe_input_keyword rikel1_0 
    where
        rikel1_0.recipe_seq=?
```

## <span style="color:#802548">_最適化：性能のための ＠EntityGraph_</span>

- 冗長なクエリがないようにEntityGraphアノテーションを使います。

```java
@EntityGraph(attributePaths = {"recipeOutputEntity", "recipeInputKeywordEntityList", "userEntity"})
@Query("""
    SELECT DISTINCT r FROM RecipeEntity r
    JOIN r.recipeInputKeywordEntityList k
    WHERE r.userEntity.userSeq = :userSeq
    AND r.recipeOutputEntity IS NOT NULL
""")
Page<RecipeEntity> findRecipesWithPagination(
    @Param("userSeq") Long userSeq,
    Pageable pageable
);
```

- 冗長なクエリはなくなりましたが、JOINがLEFTのままになっています。
- 常にデータが存在するテーブルなので、LEFTではなくINNERにすればパフォーマンスが向上します。

```sql
select
    distinct re1_0.recipe_seq,
    re1_0.created_at,
    rikel1_0.recipe_seq,
    rikel1_0.keyword,
    roe1_0.recipe_output_content_seq,
    roe1_0.output_content,
    roe1_0.recipe_seq,
    roe1_0.recipe_title,
    ue1_0.user_seq,
    ue1_0.created_at,
    ue1_0.is_deleted,
    ue1_0.roles,
    ue1_0.updated_at,
    ue1_0.user_email,
    ue1_0.user_id,
    ue1_0.user_name,
    ue1_0.user_password 
from
    recipe re1_0 
join
    recipe_input_keyword rikel1_0 
        on re1_0.recipe_seq=rikel1_0.recipe_seq 
left join
    recipe_output_content roe1_0 
        on re1_0.recipe_seq=roe1_0.recipe_seq 
left join
    user ue1_0 
        on ue1_0.user_seq=re1_0.user_seq 
where
    re1_0.user_seq=? 
    and roe1_0.recipe_output_content_seq is not null 
order by
    re1_0.created_at

Hibernate: 
select
    count(distinct re1_0.recipe_seq) 
from
    recipe re1_0 
join
    recipe_input_keyword rikel1_0 
        on re1_0.recipe_seq=rikel1_0.recipe_seq 
left join
    recipe_output_content roe1_0 
        on re1_0.recipe_seq=roe1_0.recipe_seq 
where
    re1_0.user_seq=? 
    and roe1_0.recipe_output_content_seq is not null
```


## <span style="color:#802548">_最適化：＠EntityGraph と INNERJOIN_</span>

- isNotNull も不要な条件です。レシピを作成する際には、必ず recipe_output_content がレシピと一緒に生成されるためです。isNotNullは消します。
- 削除する一方で、recipeOutputEntity はJOIN条件として保持する必要があるため、JOIN句に追加します。
- JOIN句が紛らわしくなる恐れがあるため、INNER JOIN と明示しておきます。

```java
@EntityGraph(attributePaths = {"recipeOutputEntity", "recipeInputKeywordEntityList", "userEntity"})
@Query("""
    SELECT DISTINCT r FROM RecipeEntity r
    INNER JOIN r.recipeInputKeywordEntityList k
    INNER JOIN r.recipeOutputEntity j
    WHERE r.userEntity.userSeq = :userSeq
""")
Page<RecipeEntity> findRecipesWithPagination(
    @Param("userSeq") Long userSeq,
    Pageable pageable
);
```

- recipeOutputContent テーブルは LEFTJOIN から INNERJOIN に変えました。

```sql
select
    distinct re1_0.recipe_seq,
    re1_0.created_at,
    rikel1_0.recipe_seq,
    rikel1_0.keyword,
    roe1_0.recipe_output_content_seq,
    roe1_0.output_content,
    roe1_0.recipe_seq,
    roe1_0.recipe_title,
    ue1_0.user_seq,
    ue1_0.created_at,
    ue1_0.is_deleted,
    ue1_0.roles,
    ue1_0.updated_at,
    ue1_0.user_email,
    ue1_0.user_id,
    ue1_0.user_name,
    ue1_0.user_password 
from
    recipe re1_0 
join
    recipe_input_keyword rikel1_0 
        on re1_0.recipe_seq=rikel1_0.recipe_seq 
join
    recipe_output_content roe1_0 
        on re1_0.recipe_seq=roe1_0.recipe_seq 
left join
    user ue1_0 
        on ue1_0.user_seq=re1_0.user_seq 
where
    re1_0.user_seq=? 
order by
    re1_0.created_at
Hibernate: 
select
    count(distinct re1_0.recipe_seq) 
from
    recipe re1_0 
join
    recipe_input_keyword rikel1_0 
        on re1_0.recipe_seq=rikel1_0.recipe_seq 
left join
    recipe_output_content roe1_0 
        on re1_0.recipe_seq=roe1_0.recipe_seq 
where
    re1_0.user_seq=? 
```

- 他のテーブルは INNER JOIN になりましたが、user テーブルだけは LEFT JOIN になっています。
- ユーザーが退会しただけでなく、データベースから完全に削除された場合でもレシピを表示する必要があるのであれば、ユーザーが NULL であってもデータを取得できる必要があります。したがって、LEFT JOIN が適切です。
- ただし、マイページで表示する情報は、ユーザーがログインしていなければアクセスできないページなので、LEFT JOIN を使うと無駄が多く、パフォーマンスが低下する可能性があります。
- ログイン済みのユーザーが前提となっているため、ここではEntityGraphからUserエンティティを削除し、INNER JOINに変更します。

```java
@EntityGraph(attributePaths = {"recipeOutputEntity", "recipeInputKeywordEntityList"})
@Query("""
    SELECT DISTINCT r FROM RecipeEntity r
    INNER JOIN r.recipeInputKeywordEntityList k
    INNER JOIN r.recipeOutputEntity j
    INNER JOIN r.userEntity u
    WHERE u.userSeq = :userSeq
""")
Page<RecipeEntity> findRecipesWithPagination(
    @Param("userSeq") Long userSeq,
    Pageable pageable
);
```


- それではUserテーブルの LEFT JOIN も INNER JOINに変更されます。
- INNER JOIN の方が LEFT JOIN よりも処理対象のデータが少なくなるため、クエリの実行効率が高くなる傾向があります。

```sql
select
    distinct re1_0.recipe_seq,
    re1_0.created_at,
    rikel1_0.recipe_seq,
    rikel1_0.keyword,
    roe1_0.recipe_output_content_seq,
    roe1_0.output_content,
    roe1_0.recipe_seq,
    roe1_0.recipe_title,
    re1_0.user_seq 
from
    recipe re1_0 
join
    recipe_input_keyword rikel1_0 
        on re1_0.recipe_seq=rikel1_0.recipe_seq 
join
    recipe_output_content roe1_0 
        on re1_0.recipe_seq=roe1_0.recipe_seq 
join
    user ue1_0 
        on ue1_0.user_seq=re1_0.user_seq 
where
    ue1_0.user_seq=? 
order by
    re1_0.created_at

select
    count(distinct re1_0.recipe_seq) 
from
    recipe re1_0 
join
    recipe_input_keyword rikel1_0 
        on re1_0.recipe_seq=rikel1_0.recipe_seq 
join
    recipe_output_content roe1_0 
        on re1_0.recipe_seq=roe1_0.recipe_seq 
join
    user ue1_0 
        on ue1_0.user_seq=re1_0.user_seq 
where
    ue1_0.user_seq=?
```

## <span style="color:#802548">_最適化： DISTINCT 削除_</span>

- しかし、JPQL ではなく NATIVE SQL としては DISTINCT が不必要です。
- OneToOneやOneToManyなどのエンティティ間の関連を表すアノテーションによって JPA で発生した重複を除外するために JPQL で DISTINCT が必要だけです。


```java
public Page<RecipeMyPageResponse> findAllRecipeByUser(Long userSeq, int currentPage) {
    Pageable pageable = PageRequest.of(currentPage, 3, Sort.by(Sort.Direction.ASC, "createdAt"));
    
    return recipeMyPageRepository.findRecipesWithPagination(userSeq, pageable)
                                    .map(RecipeMyPageResponse::toDTO);
};
```

- DISTINCT を削除して JAVA 側で 処理するようにします。 
- NATIVE SQLで DISTINCT は削除されます。不必要の作動がなくなるので性能が高くなります。
- 重複を削除するためページネーションなので順序が重要なのでLINKED

```java
@EntityGraph(attributePaths = {"recipeOutputEntity", "recipeInputKeywordEntityList"})
@Query("""
    SELECT r FROM RecipeEntity r
    INNER JOIN r.recipeInputKeywordEntityList k
    INNER JOIN r.recipeOutputEntity j
    INNER JOIN r.userEntity u
    WHERE u.userSeq = :userSeq
""")
Page<RecipeEntity> findRecipesWithPagination(
    @Param("userSeq") Long userSeq,
    Pageable pageable
);
```

- Javaで重複を削除する方法について、2つの工夫を行いました。
- 最初は、セットクラスを使ってロジックを実装しました。
    - LinkedHashSetの重複判定は、ハッシュコードを基準に行われます。
    - そのため、Recipeエンティティに@EqualsAndHashCodeを使用しました。
    - ただし、重複の基準はrecipeSeq変数のみであるため、@EqualsAndHashCodeにその点を明示的に指定しました。

```java
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Entity
@Table(name = "recipe")
@EqualsAndHascode(of = "recipeSeq")
public class RecipeEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "recipe_seq")
    private Long recipeSeq;
    .
    .
}

public Page<RecipeMyPageResponse> findAllRecipeByUser(Long userSeq, int currentPage) {
    Pageable pageable = PageRequest.of(currentPage, 3, Sort.by(Sort.Direction.ASC, "createdAt"));
    Page<RecipeEntity> rawPage = recipeMyPageRepository.findRecipesWithPagination(userSeq, pageable);

    List<RecipeEntity> deduplicated = new ArrayList<>(new LinkedHashSet<>(rawPage.getContent()));

    List<RecipeMyPageResponse> dtoList = deduplicated.stream()
                                                        .map(RecipeMyPageResponse::toDTO)
                                                        .toList();

    return new PageImpl<>(dtoList, pageable, rawPage.getTotalElements());
};
```

- DISTINCT がない場合は COUNTクエリを明確にして暗黙的なクエリを呼び出さないように処理します。

```java
@EntityGraph(attributePaths = {"recipeOutputEntity", "recipeInputKeywordEntityList"})
@Query(value = """
    SELECT r FROM RecipeEntity r
    INNER JOIN r.recipeInputKeywordEntityList k
    INNER JOIN r.recipeOutputEntity j
    INNER JOIN r.userEntity u
    WHERE u.userSeq = :userSeq
""",  countQuery = """
        select count(r) FROM RecipeEntity r
        where r.userEntity.userSeq = :userSeq
""")
Page<RecipeEntity> findRecipesWithPagination(
    @Param("userSeq") Long userSeq,
    Pageable pageable
);
```

- しかし、単にページネーションのためだけにエンティティクラスに @EqualsAndHashCode を付け加えるのは好ましくないと感じました。
- セットクラスとマップクラスは、内部実装を確認したところ、CPU性能にそれほど大きな違いはないように思われたため、マップクラスによる実装も検討しました。
- マップは、hashCode や equals に依存せずに処理できる点が利点です。
- ページネーションでは順序が重要となるため、LinkedHashMap を使用しました。

```java
public Page<RecipeMyPageResponse> findAllRecipeByUser(Long userSeq, int currentPage) {
    Pageable pageable = PageRequest.of(currentPage, 3, Sort.by(Sort.Direction.ASC, "createdAt"));

    Page<RecipeEntity> rawPage = recipeMyPageRepository.findRecipesWithPagination(userSeq, pageable);

    List<RecipeEntity> deduplicatedList = rawPage.getContent().stream()
                                                .collect(Collectors.toMap(
                                                    RecipeEntity::getRecipeSeq,
                                                    Function.identity(),
                                                    (a, b) -> a,
                                                    LinkedHashMap::new
                                                ))
                                                .values()
                                                .stream()
                                                .toList();

    List<RecipeMyPageResponse> response = deduplicatedList.stream()
                                                        .map(RecipeMyPageResponse::toDTO)
                                                        .toList();

    return new PageImpl<>(response, pageable, rawPage.getTotalElements());
}
```




## <span style="color:#802548">_最適化： 大容量データに備えて SQL 切り替え_</span>


- 上のSQLは一回で全部データを受け取ることができますが、JOIN の数が多いです。
- それを JOIN が少なくする必要があります。
- SQL を二回に分けて大容量のデータを受け取るようにします。
- 二回に分けて性能が悪くなるのではないかと思うかもしれません。

```java
@Query("""
        SELECT r.recipeSeq FROM RecipeEntity r
        WHERE r.userEntity.userSeq = :userSeq
    """)
    List<Long> findRecipeIdsByUser(@Param("userSeq") Long userSeq);

    @EntityGraph(attributePaths = {"recipeOutputEntity", "recipeInputKeywordEntityList"})
    @Query(value = """
        SELECT r FROM RecipeEntity r
        INNER JOIN r.recipeInputKeywordEntityList k
        INNER JOIN r.recipeOutputEntity j
        WHERE r.recipeSeq IN :recipeIds
        and r.isDeleted = false
    """, countQuery = """
            select count(r) FROM RecipeEntity r
            WHERE r.recipeSeq IN :recipeIds
            and r.isDeleted = false
    """)
    Page<RecipeEntity> findRecipesByIds(@Param("recipeIds") List<Long> recipeIds, Pageable pageable);
```

- ただし、大容量のデータを扱う場合は、JOIN を減らすことでデータベースが照会する量も少なくなるので、応答性を高めることができます。

```java
public Page<RecipeMyPageResponse> findAllRecipeByUser(Long userSeq, int currentPage) {
    Pageable pageable = PageRequest.of(currentPage,  10, Sort.by(Sort.Direction.DESC, "createdAt"));
    
    List<Long> recipeIds = recipeMyPageRepository.findRecipeIdsByUser(userSeq);
    Page<RecipeMyPageResponse> recipes = recipeMyPageRepository.findRecipesByIds(recipeIds, pageable).map(RecipeMyPageResponse::toDTO);

    return recipes;
};
```

