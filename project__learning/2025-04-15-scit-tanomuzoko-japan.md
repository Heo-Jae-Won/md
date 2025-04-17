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


## <span style="color:#802548">_js closure_</span>

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


## <span style="color:#802548">_最適化_</span>

- コードレビューをしながらいくつかの修正しなければならないことを見つけました。



