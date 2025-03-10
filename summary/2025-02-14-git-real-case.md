## <span style="color:#802548">_git fork 후 PR_</span>
- pull request하는 방법은 아래에 잘 서술되어 있다.  

https://wayhome25.github.io/git/2017/07/08/git-first-pull-request-story/

## <span style="color:#802548">_git clone 특정 branch_</span>

```sh
git clone -b(--branch) <branchname> <remote-repo-url>
# default가 아닌 특정 branch를 clone

# ex) git clone -b dev https://github.com/fff/xxx.git
```

## <span style="color:#802548">_git clone시 권한 에러_</span>
- git clone https://github.com/id/reponame.git와 같이 했는데 오류가 났다.

```
fatal: repostiroy not found가 뜰 경우
```

- 그 경우 git clone https://userid@github.com/id/reponame.git와 같이 앞에 username을 붙여준다.
- 그리고 나면 해당 유저로 로그인하여 증명하라는 표시가 뜬다. 로그인하여 증명하면 clone이 가능하다.
- 기본적으로 해당 오류가 나는 것은, 권한을 얻지 못했기 때문이다. 
- private repo에 접근하려는데 권한이 없는 경우 자주 발생한다. 따라서 해당 아이디로 로그인해서 credentials manager를 얻는 것이다.




## <span style="color:#802548">_git clone만 한 경우 다른 branch로 이동_</span>
- 혹시 까먹고 branch를 특정하지 않고 master(혹은 main)로 가져왔어도 상관없다.
- 보통 github clone으로 가져온경우, CLI 상으로 main만이 local repo에서 보인다.

```sh
git branch -r
# remote에 어떤 branch가 있는지 보여줌
git switch [이동하고자 하는 branch]
# upstream과 자동으로 tracking을 활성화하며 local에 해당 branch를 만듦.

# ex) git switch feature/board
```

## <span style="color:#802548">_git remote에 새로 추가된 branch를 인식_</span>
- proejct leader가 remote에서 만든 branch로만 진행하는 경우, 필요하다.
- remote에서 branch를 만든다고 해도, local에서는 인식이 안된다.
- 아래 명령어로 git config 파일을 동기화 시켜줘야 한다.

```sh
git fetch origin
# remote에 update 된 branch를 인식
git branch -r 
# remote에 추가된 branch들이 노출됨. remote update 전에는 안 보임.
```

## <span style="color:#802548">_remote 설정_</span>
- clone으로 받아왔다면 remote가 이미 설정되어 있어 상관없다.
- remote는 보통 이름을 origin으로 정한다.

```sh
git remote add [저장소 alias] [url] 
# 원격저장소를 설정한다.

# ex) git remote add origin https://github.com/fff/xxx.git

git remote -v :
# 설정된 원격저장소와 저장소이름 정보 출력

git remote set-url [저장소 alias] [url] : 
# 기존 원격저장소를 새로운 원격저장소로 바꿈

# ex) git remote set-url origin https://github.com/aaa/xxx.git

git remote remove [저장소 alias] : 
# 원격저장소와 연결 끊으며, 저장소 alias도 사라진다.

# ex) git remote remove origin
```


## <span style="color:#802548">_git config setting_</span>

```sh
git config --list 
# 설정해둔 변수들을 확인한다.

git config --global user.name "유저이름" 
# 해당 유저이름으로 commit이 남게 설정한다.

git config --global user.email "이메일" 
# 해당 이메일로 commit이 남게 설정한다.

git config --unset 속성  
# 해당 속성의 값을 삭제한다.

#ex ) git config --unset user.name

git config --global init.defaultbranch main
# default branch의 이름을 master가 아닌 main으로 한다
```

## <span style="color:#802548">_git config 안되는 오류_</span> 
- git을 깔고나서 config를 설정하려 할 때 오류가 나는 경우가 있다.

```
could not lock config file error
```

- 그럴 땐 처음에 bash 관리자 권한 실행하고 아래와 같이 주었다.

```
git config --system --unset credential.helper
```

- 그랬더니 해당 오류는 일어나지 않았다.
- 하지만 근본 원인은 HOME 경로였다.
- git은 시스템 변수를 HOME으로 만들어서 사용하는데, 이미 시스템 변수에 HOME을 JDK 경로로 등록해버려서 생기는 오류였다.
  - git은 HOME 시스템변수($HOME) 으로 자신의 경로를 설정한다.
  - 그런데 HOME 시스템변수에 java 경로륾 명시적으로 박았다.
  - 그러니 git 경로가 git/cmd가 아니라 java 경로가 되어버린 것이다.
  - 이런 오류 떄문에 HOME이 아닌 JAVA_HOME, MAVEN_HOME 같은 형태로 시스템 변수를 만드는 것이었다.
- 해당 경로에 gitconfig 파일이 없으니 keygen 혹은 git config 시에 오류가 나게 되는 것이다.


## <span style="color:#802548">_branch_</span>

```sh
git switch -c [branch이름] 
# local에 해당 branch를 만들면서 이동까지 동시에 함
git branch -d [branch이름]
# local에서 해당 branch를 제거
git branch -m [바꿀branch이름] [바뀐branch이름]
# local에 만든 branch 이름을 변경
git branch -r 
# 원격의 branch만 보여줌
git branch
# local의 brtanch만 보여줌
git branch -a 
# 원격과 local branch 전부 보여줌
```

## <span style="color:#802548">_git remote-local branch 동기화_</span> 
- remote에 없지만 local에만 있는 것은 일괄로 지워버릴 수 있다.
- 다만 그것은 remote와 연결되어 있는 것들에 관해서만 가능한 것이다.
- 즉, remote에 원래 ref가 있었는데, 지금은 사라진 것들만 대상이다. 
- 다시 말하면 remote와 연결된 적도 없는 것들은 지워지지 않고 남아있다.

```sh
git fetch --all --prune
```



## <span style="color:#802548">_branch 이동에 필요한 stash_</span>
- 소스코드에 변화가 있다고 가정해보자.
- 변화를 가지고 git switch branch하면 자연스럽게 넘어가지는 경우도 있다.
  - 이동하려는 branch가 fast-forward가 가능하다면 git add를 하지 않아도 branch switch가 가능하다.
- 그런데 아래와 같은 오류 메시지가 뜨는 경우도 있다.

```
error: Your local changes to the following files would be overwritten by checkout:
        src/main/resources/15125125
Please commit your changes or stash them before you switch branches.
Aborting
```

- 만약 conflict가 일어날 것이 분명하다면, git switch가 불가능하다기에 반려시키는 것이다.
- 그럴 때 git stash를 사용해 local changes를 저장한다. 
- local 작업파일 임시 저장한다는 의미다.
- 특히 실수로 다른 branch에서 작업한 내용을 내 branch로 옮길 때 유용하다.

```sh
git stash
git switch [원래 branch]
git stash apply
```

## <span style="color:#802548">_pull에 필요한 stash_</span>

- 그 외에도 pull을 받을 때 자신이 작업해 놓은 파일로 인해 에러가 나는 경우도 있다. 위와 똑같은 에러가 난다.
- 그 경우에도 stash를 하고 pull을 받은 다음에 stash로 임시 저장한 작업 파일들을 다시 불러온다.

```sh
git stash
git pull origin main
git stash apply
```


## <span style="color:#802548">_hotfix에 필요한 stash_</span>

- remote가 아닌 local에서 branch를 파서 remote에 올려야 하는 경우도 있다.
- 예를 들어 hotfix라고 쳤을 떄, hotfix 작업 내용을 진행하다가 나중에 branch를 hotfix로 바꾸지 않았다는 것을 알았다.
- 그 경우는 아래와 같이 적용해준다.

```sh
git stash # hotfix 이전까지 진행하던 내용들만
hotfix 코드 작성 # 까먹고 branch 안 바꿈
git stash # stash list에 2개가 생긴 것
git stash branch [branch이름] # branch 만들고 이동하면서 hotfix 내용도 가져감. stash list에서 drop되서 사라짐
소스코드 작성 후 git add commit push PR
git stash list # 다른 stash 안 했다는 가정 하에
git switch [원래 branch]
git stash apply 
git stash drop # conflict 없이 제대로 apply 되면
```

- 그 뒤 hot fix branch는 지워주자. hotfix는 단발적으로 사용하는 branch다. 
- local에서도, remote에서도 지우자.
  - 물론 이도 회사 branch 관리 전략에 따라 다르니 상사한테 물어보고 지우자.

```sh
git push origin --delete [branch이름] # local과 remote 모두 지우기
git branch -d [branch이름] # local에서만 branch 지윅
```


## <span style="color:#802548">_코드를 가지고 놀 떄 필요한 stash_</span>

- 그외 버그를 해결할 때나 다양한 방법으로 코드를 구현해볼 때, 즉 여러 시도를 할 때도 git stash를 여러번하여 실패한 버전과 실패했던 이유를 떠올릴 수 있다.
- 이 경우에는 stash를 할 때 이름을 주면서 stash를 시도한다.

```sh
git stash save "PageRequest를 이용한 pagination의 구현" (PageRequest로 구현하고서)
git stash save "Cursor를 이용한 pagination의 구현"   (Cursor 방식으로 구현하고서)
git stash save "ajax를 이용한 pagination의 구현"    (ajax로 구현하고서)
git stash save "fecth를 이용한 pagination의 구현"  (fetch로 구현하고서)
```

- 그 경우 아래처럼 생긴다

```sh
stash@{0}: On hotfix: PageRequest를 이용한 pagination의 구현
stash@{1}: On hotfix: Cursor를 이용한 pagination의 구현
stash@{2}: On hotfix: ajax를 이용한 pagination의 구현
stash@{3}: On hotfix: fecth를 이용한 pagination의 구현
```

- 구현한 것 중 성능, UX 등을 고려하여 최적이라 생각되는 것만 git stash apply 한 뒤 add commit push한다.
  - 물론 시간 남을 떄 이렇게 하는 것이다. 일단 기능 구현부터 해내야 한다.
  - stash가 남아있으면 commit history가 더러우니까 clear로 지워주자.

```sh
git stash apply stash@{2}
git add
git commit
git push
git stash clear
```

- 파일 한 개만 stash 하고 싶다면, 아래와 같은 형태로 stash를 해주면 된다.

```sh
git stash push -m "message" [path]
git stash push -m "info_mypage.js closure" src/main/resources/static/js/info_mypage.js
```

## <span style="color:#802548">_stash가 되지 않는 대상_</span>

- 처음 만들어져서 commit 되지 않은 파일들은 stash의 대상이 되지 않는다.
- git switch를 할 때 처음만든 것과 commit된 것이 섞여있는 경우가 문제다.
  - 이 경우 git stash를 하면 cannot resolve name이 일어날 수 있다.
  - commit된 파일의 변경사항은 stash가 되었는데, 새로 만든 파일은 stash가 되지 않는다.
  - 따라서 새로 만든 파일에서 stash된 파일의 변경사항을 참조하고 있다면 error가 뜬다.

- 이럴 떈 commit을 하던가, 그냥 노가다로 파일을 밖에 보존시켜놓는 게 답이다.
- 아무리 stash를 해도 처음 만든 대상은 stash되지 않는다.

## <span style="color:#802548">_stash 복구하기_</span>
- 작업하던 것을 stash해놨는데, 실수로 clear 했을 때가 있다.
- 그럴 때는 아래 명령어를 사용하여 clear 된 stash를 확인한다.

```sh
git fsck --no-reflog | awk '/dangling commit/ {print $3}' | xargs -L 1 git --no-pager show -s --format="%ci %H" | sort
```

- vscode의 gitlens가 깔렸다는 가정하에, 
  -  git stash를 클릭해보면서 자기가 만들었던 파일이 포함되어 있는지 확인해준다.
  - stash이므로 WIP 같은 이름으로 되어있을 것이다.
  - WIP 같은 이름이 아닌 것들은 무시하자.
  - WIP가 붙어있는 것들은 files changed를 보면서 바뀐게 내것이 맞는지 확인하자.


```
2025-02-09 21:44:35 +0900 2e2c8c25a11e511dbe70952d8d7700720ed5d0fa
2025-02-10 22:08:17 +0900 31444f3e87ccad128ef5f8df741e0b713f194532
2025-02-14 08:39:32 +0900 bc6ac83ff0be90b9292bdf09a63393e740dd8060
2025-02-14 08:40:21 +0900 63a4c85fbb6423f95f472cbdbe7a516fa5d57e4c
2025-02-14 08:40:50 +0900 f6b7e5b7d02969e3043eb0e23dc43486db31b5e6
2025-02-14 09:05:57 +0900 f4d6dcc2524b76044a3d4c586dea7739856b45b8
2025-02-14 09:07:17 +0900 21d23eb2bc7c492d3f818bb8a5326363f518d626
2025-02-14 09:07:32 +0900 7e2ad011a1122a07998cf739d129fc19ee95489a
2025-02-14 09:54:32 +0900 d6dd20cfa29ed8038719604dceefca497ed752d4
2025-02-14 09:54:45 +0900 c7375a29886d86868b183c673e7e4e6649e2dc4b
2025-02-14 09:54:53 +0900 1f0c61aaecbfe69d7eb282918c1d5d38f4c0207b
2025-02-14 09:55:03 +0900 5c90d6ef9e44d2a8b0c6642632c4321261d72523
2025-02-14 10:04:53 +0900 f94298348008956c10e1fc7f07fa8aa1e9b52d33
2025-02-14 10:07:03 +0900 1bdd99f315d1766ea1dd99b170b1aa675fca951a
2025-02-14 10:09:49 +0900 16303e59d536347dc47c1b1bea68d933671436bb
2025-02-14 10:11:16 +0900 aa2863429a09b3db8432e20c8046c8f4b6029a8e
2025-02-14 15:38:08 +0900 acdb60f812a35ff1f004c93fbed0dbdab5259f73
2025-02-14 15:41:36 +0900 b92bdb8eef01bcbe58112e61c362025c4316d54a
2025-02-14 15:43:56 +0900 d0f500d614f71bbd808d9dac03bbd7eb362ba3be
2025-02-14 16:44:49 +0900 b546bb28f42d3799749a753b39cfacff0892f78d
2025-02-14 17:03:31 +0900 d30bbce15ca7049271f7fab35c5c23d88bc380b7
2025-02-14 17:04:50 +0900 f90bb3abe2140f1ecf949d288b8439fa103568fc
2025-02-14 17:06:58 +0900 bb16264d5fec3bcb49f4231f69f63ffd15a7b27e
2025-02-14 20:29:30 +0900 4f8bfa9eebc0659c3c456cf6973de8d30c0d6112
2025-02-14 20:29:34 +0900 9ff0c28b311ca8ebe90be51c90a45698b57a4869
2025-02-14 20:29:38 +0900 9cb4b485f7390fc833a19e6e8999abb3bff23c10
2025-02-14 20:29:43 +0900 55bd4620460ab803208f3c3043309cab1c40082d
```

- 그래서 자기가 만든 파일들이 맞으면 아래와 같이 사용한다.

```sh
git stash apply [commithash값]
```


## <span style="color:#802548">_staging_</span>
- class명만 딱 인식하는 방법이 없어서 번거롭다.
- GUI로 하면 변화 사항도 같이 보여준다.
- 엔간하면 GUI로 하도록 하자.

```sh
git add -u 
# 수정하고 삭제한 것들만 반영하며, 새로 추가된 파일은 staging area에 반영되지 않는다. 
git add . 
# 수정, 삭제, 추가된 파일들을 전부 반영한다. git ignore에 있는 파일은 제외된다. 만약 git add를 하고 싶지 않다면 아래에서 볼 git stash를 이용한다. 
git add [file이름]
#ex) git add src/main/java/org/scit/project/MainController.java
```

## <span style="color:#802548">_unstaging_</span>
- staging과 마찬가지 이유로 엔간하면 GUI로 하도록 하자.

```sh
git restore --staged . 
# git add한 것을 모두 staging area에서 제외시킨다.

git resotre --staged [fileName]
# 해당 파일만 staging에서 제외

#ex) git restore --staged src/main/java/org/scit/project/MainController.java

```

## <span style="color:#802548">_local change 지우기_</span>
- 아직 staging에 올라오지 않은 것은 아래와 같이 지울 수 있다.
- 대신 지우면 완전히 날라간다. 그러니 조심하자. 꼭 필요한 지 고민해서 쓰자.

```sh
git restore [filename]
```

- 한꺼번에 다 지우고 싶다면?

```sh
git restore .
```

- 만약 파일 내용을 전전 commit으로 돌리고 싶다면? 그러면서 HEAD는 유지하고 싶다면?


```sh
git restore --source HEAD~2 [파일이름]
git restore --source [해쉬이름] [파일이름]
```

## <span style="color:#802548">_뺴먹은 내용 포함하여 다시 commit_</span>
- 직전 commit에 포함시킬 경우 사용 가능한 방법이다.
- 온전히 하나의 commit에 다 담을 수 있다.

```sh
git add [file이름] 
git commit --amend 
# commit을 했는데 빼먹은 파일이 있다면 staging area에 넣고 다시 commit함.
```

## <span style="color:#802548">_local repo의 commit 삭제_</span>
- git reset --hard는 엔간하면 쓰면 안된다.
- remote에 올라간 commit에 대해선 절대 사용하지 말자.

```sh
git reset --mixed : 
# 커밋이력을 없애면서 git add한 것들을 모두 없앤다. 소스코드는 유지된다. 

git reset --soft : 
# 커밋이력을 없애면서 여태까지 git add한 것을 보존하면서 소스코드는 유지된다. 

git reset --hard : 
# working dir과 staging area 모두에서 지운다
```
​

## <span style="color:#802548">_remote repo의 commit 삭제_</span>
- remote에 올라간 commit을 삭제하는 것은, 실제 삭제가 아니다.
- 해당 commit을 무효화하는 추가 commit을 하는 형태다.
- head로 갈 commit 이전 까지의 commit이 revert 대상이 된다.

```sh
git revert --no-commit [latest]...[HEAD로 갈 commit] : 
## 여러 개의 commit이력을 없애고자 할 때 전부에 대해 revert commit을 남기지 않고 1개의 commit만 남긴다.

git revert --no-commit HEAD~3
## 여러개를 revert 해도 commit은 1개만 남는다. 각각의 revert commit을 남기지 않는다.

git push origin master
```



## <span style="color:#802548">_commit log_</span>
- git commit 이력 보는 것은 아래가 기본이다.
- 하지만 alias를 설정해주는 게 좋다.
- git log 메시지를 이쁘게 뽑는 것은 아래 사이트를 참고하자.

```
https://www.durdn.com/blog/2012/11/22/must-have-git-aliases-advanced-examples/ 

위 사이트의 특수한 명령어는 alias를 설정한 것이고, 이는 아래에서 보는 방식으로 설정해준다.
```

- 더 자세한 git log message 옵션에 대해서는 아래 site를 참고하자.
- https://www.atlassian.com/git/tutorials/git-log

```sh
git log --oneline : 
# commit 로그를 짧게 중요한 정보만 요약해 출력함.

git log --since="2023-04-02" : 
# 해당 날짜 이후의 commit 이력만 출력한다. 

git log --author=username : 
# 해당 username이 남긴 commit 이력을 확인한다.

git log --grep="hotfix" : 
# hotfix라는 commit 메시지를 출력한다.

git log -S "change" : 
# change라는 소스코드가 들어간 commit을 출력한다.
```


## <span style="color:#802548">_git log message에서 branch 읽기_</span>
- 현재 내 작업 폴더의 current branch는 feature/authority-profile다.
- 거기서 Head commit은 785cd77이다. 하지만 remote는 안써져 있다. 
- 따라서 remote에는 더 commit이 쌓여있다는 것을 알 수 있다. 
- 다른 분이 remote에 더 넣었다면 내가 최신 commit이 아닐 수 있다.
  - git pull이 필요할 수 있다는 의미다.

```sh
785cd77 (HEAD -> feature/authority-profile) Feat: generate role
12d8f42 Feat: role list and menu tree handling
926b4c9 Feat: user role, menu, changed protocol
68aa468 (master) Merge pull request #122 
5a792bb change asset admin email
78cbf6f Merge pull request #121 
8957fea (origin/hotfix/add-db-tls) add-db-enable-tls
1ab1ef3 Merge pull request #120 
4cae66c fix: exclude deleted assets & add update date
82174d3 (feature/dev_wsg) fix: exclude deactivated account#2
8339e87 fix: exclude deactivated account
afdfd6d add: search word option(=address)
35d69ef (origin/feature/dev_wsg) 프로필 이미지 경로 추가
```

- 만약 내가 remote의 최신 commit을 가지고 있다면, HEAD가 origin도 같이 갖고 있는다.
- remote에서도 해당 commit이 최신이라는 의미다.
  - 아래 4c6cfcc라는 commit이 feature/recommend의 local, remote branch 어디든 최신이다. 
  - 반면에 main은 바로 그 commit이 최종 commit이다. 
    - origin/HEAD에서 HEAD는 head commit개념이 아니라 default branch 개념이다.

```sh
4c6cfcc (HEAD -> feature/recommend, origin/feature/recommend) commit2
de12b0e (origin/main, origin/HEAD, main) commit
```

- default branch를 보고 싶다면 아래 명령어를 실행하자.
- default branch를 바꾸려면 github와 local 양쪽에서 바꿔줘야 한다.
- 굳이 바꿀 이유가 없으니 그냥 놔두자.

```sh
git remote show origin
```

- 아래를 보면 다른 것들은 전부 origin이 branch명에 붙었는데, 안 붙어있는 게 있다.
- 이런 것들은 remote 상에 존재하지 않기 때문에 origin이 붙어있지 않은 것이다.

```sh
4cae66c fix: exclude deleted assets & add update date
82174d3 (feature/dev_wsg) fix: exclude deactivated account#2
```

- master에 origin이 안붙어있는 이유는 최신 commit이 아니라서 그런 것이다.
- pull로 최신화해줄 떄까지 origin/을 안붙여준다.

```sh
68aa468 (master) Merge pull request #122 
5a792bb change asset admin email
```



## <span style="color:#802548">_git log option을 활용한 개발기간 동안의 파일 목록 추출_</span>
​

- 특정 기간 내 개발자 작업 파일만 추출하는 명령어다.
- 다만 주의할 사항은 since의 대상은 authorDate라는 점이다.
  - authorDate는 처음 변화의 시작 시간이다.
  - commit을 한 뒤 첫 타자를 치면 그게 authorDate가 된다.
- commitDate기준이 아니다. 그 점을 헷갈리지만 않으면 된다.
  - git commit을 한 순간이 commitDate다.
  - commitDate까지 보고 싶다면 option을 주자.

```sh
git log --name-only --author=Heo-Jae-Won --date=local --since="2024-01-03" --format=fuller | egrep -v "^commit|^Date:|^Merge:|^Author:|^\s" | sort | uniq 

##
AuthorDate: Thu Jan 4 14:34:03 2024
AuthorDate: Tue Jan 9 16:54:43 2024
AuthorDate: Wed Jan 10 09:49:36 2024
Commit:     Heo-Jae-Won <heojaewon@xx>
CommitDate: Mon Jan 8 17:16:11 2024
CommitDate: Tue Jan 9 16:56:41 2024
CommitDate: Wed Jan 10 09:53:40 2024
src/main/java/net/xx/zz/map/service/MapService.java
src/main/java/net/xx/zz/user/repository/UserRepository.java
src/main/java/net/xx/zz/user/specifications/UserSpecifications.java
##
```

- 특정 저자와 날자부터의 commit 메시지 및 동반된 파일 수정 목록을 보여줌

```sh
git log --name-only --date=local --author=Heo-Jae-Won --since="2024-01-03"

##
commit 82174d3e035a961cad648f9775c68235570b1219 (feature/dev_wsg)
Author: Heo-Jae-Won <heojaewon@xx.net>
Date:   Wed Jan 10 09:49:36 2024

    fix: exclude deactivated account#2

src/main/java/net/xx/zz/map/service/MapService.java
src/main/java/net/xx/yy/user/repository/UserRepository.java
src/main/java/net/xx/yy/user/specifications/UserSpecifications.java

commit 8339e87f0f849018ed1bbabb6e3d99b20506e9fa
Author: Heo-Jae-Won <heojaewon@xx.net>
Date:   Tue Jan 9 16:54:43 2024

    fix: exclude deactivated account

src/main/java/net/xx/yy/map/service/MapService.java
src/main/java/net/xx/yy/user/repository/UserRepository.java

commit afdfd6d52b58eac8f73731336ff5879a89e687c6
Author: Heo-Jae-Won <heojaewon@xx.net>
Date:   Thu Jan 4 14:34:03 2024

    add: search word option(=address)

src/main/java/net/xx/yy/map/service/MapService.java
src/main/java/net/xx/yy/user/repository/UserRepository.java
##
```

- 특정 저자와 날자부터의 commit 주제행 메시지 및 동반된 파일 수정 목록을 보여줌.

```sh
git log --name-only --author=Heo-Jae-Won --date=local  --since="2024-01-03" --no-merges | egrep -v "^commit|^Author|^Date"

##

    fix: exclude deactivated account#2

src/main/java/net/xx/yy/map/service/MapService.java
src/main/java/net/xx/yy/user/repository/UserRepository.java
src/main/java/net/xx/yy/user/specifications/UserSpecifications.java


    fix: exclude deactivated account

src/main/java/net/xx/yy/map/service/MapService.java
src/main/java/net/xx/yy/user/repository/UserRepository.java


    add: search word option(=address)

src/main/java/net/xx/yy/map/service/MapService.java
src/main/java/net/xx/yy/user/repository/UserRepository.java
##
```

- 특정 저자와 날짜부터의 commit 파일목록만 보여준다.
- commit이 겹치는 것들은 전부 빼고 보여준다.

```sh
git log --name-only --author=Heo-Jae-Won --date=local --since="2024-01-03" | egrep -v "^commit|^Date:|^Merge:|^Author:|^\s" | sort | uniq


##
src/main/java/net/xx/yy/map/service/MapService.java
src/main/java/net/xx/yy/user/repository/UserRepository.java
src/main/java/net/xx/yy/user/specifications/UserSpecifications.java
##
```



## <span style="color:#802548">_command 별칭 설정하기_</span>
- ​gitconfig는 git에서 인식 가능한 command만을 alias로 설정할 수 있다. 
- 다른 system, 예를 들어 pnpm의 경우, 또 다른 system이라서 인식되지 않는다.
- 저기다가 써도 alias로서 작동하지 못한다.
- 아래처럼 우선 gitconfig를 연다.

```sh
vim ~/.gitconfig: 
```

- 위 명령어를 클릭하면 처음에는 user 정보 정도만 노출될 것이다.
- alias는 아래처럼 써준다.

```sh
[user]
        email = ckdwhgood2@naver.com
        name = Heo-Jae-Won
[alias]
        ls = log --pretty=format:"%C(yellow)%h%Cred%d\\ %Creset%s%Cblue\\ [%cn]" --decorate
        ll = log --pretty=format:"%C(yellow)%h%Cred%d\\ %Creset%s%Cblue\\ [%cn]" --decorate --
        lo = log --oneline
```

- git config --list로 쳐보면 설정한 email, name, alias가 잘 들어온 것을 확인할 수 있다.

```sh
alias.ls=log --pretty=format:%C(yellow)%h%Cred%d\ %Creset%s%Cblue\ [%cn] --decorate
alias.ll=log --pretty=format:%C(yellow)%h%Cred%d\ %Creset%s%Cblue\ [%cn] --decorate --numstat
```



## <span style="color:#802548">_commit message 바꾸는 rebase -i_</span>
- remote에 올라가지 않는 commit의 경우에만 적용 가능하다.
- remote에 올라간 commit은 어쩔 수 없지만 그냥 놔두자.
- 아래처럼 commit이 있다고 가정해보자. 아직 local에만 있는 commit이다.

```sh
7e2ad01 (HEAD) commit2
c236529 commit1
67afe4d s is added
5955146 Merge pull request #1 from scit-project-46-B-5/main
555313d init project
82094a2 Initial commit
```

- 만약 c236529 commit의 message를 바꾸고 싶다면?
- rebase -i 는 head를 해당 commit hash로 잡는다는 의미다.
  - 따라서 head를 c236529로 잡으면 c236529 commit message는 수정 불가다.
  - 거기서 한 번 더 내려가주자. head를 67afe4d로 옮겨야 c236529의 commit을 바꿀 수 있다.

```sh
git rebase -i 67afe4d   # c236529로 하면 안되고 이거보다 한 칸 더 내려가야 한다. 
pick을 reword로 변경하고 :wq 
commit message를 변경하고다시 :wq
```

- rebase기 때문에 rebase되는 2개의 commit은 hash값이 다 바뀐다.
- 만약에 다른 사람들에게 공유된 commit이었다면, 내용은 같아도 commit hash가 달라 다른 commit 취급된다.
- 그럼 history가 달라져 conflict가 나게될 확률이 매우 높아진다. 
- 따라서 이미 push되고, 다른 사람에게 공유된 commit은 rebase 금지다.

```sh
55245fs (HEAD) commit2
bvbaf1z d is added
```

## <span style="color:#802548">_commit merge용 rebase -i_</span>

- 커밋 병합도 가능하다.
  - key properties를 넣는 commit을 했는데, 실수로 안넣어서 두번째 commit을 했다고 해보자.
  - 그럴 때 병합을 하는데, fixup을 사용한다.
  - 없어진 commit의 내용은 그 다음 commit에 그대로 녹여진다.
- 오타를 낸 걸 수정하고, 또 수정하는 commit을 했을 때도 유용하게 쓸 수 있다.
- fixup을 2개를 하면, 2개 commit이 바뀐 내용을 합쳐서 다음 commit에 녹여낸다.
- 아래와 같은 commit history가 있다고 해보자.


```sh
q5141414 my cat made this commit
e151241 fix another navbar typo
r151295 fix navbar typos
```

- 아래와 같은 명령어를 사용한다.
- index 단위라서 3이면 0, 1, 2nd index로 3개의 commit을 바꿀 수 있다.


```sh
git rebase -i HEAD~3
```

-  그럼 아래와 같이 vim이 열린다.


```sh
pick q141414 my cat made this commit
pick e151241 fix another navbar typo
pick r151295 fix navbar typos
```


- 그럼 아래 오타 커밋을 fixup으로 바꿔준다.


```sh
pick q141414 my cat made this commit
fixup e151241 fix another navbar typo
fixup r151295 fix navbar typos
```

- 그럼 아래와 같이 commit history가 변경된다.

```sh
v51241 my cat made this commit
```

## <span style="color:#802548">_commit 삭제용 rebase -i_</span>

- commit을 아예 삭제하는 것도 가능하다.
- 똑같이 현재 branch를 rebase 해준다.


```sh
git rebase -i HEAD~4
```

- 그럼 아래와 같이 열린다.


```sh
pick k51251 my father made this commit
pick v51241 my cat made this commit
pick e14124 not useful commit
pick g15155 initial commit
```


- 여기서 not useful commit을 drop으로 바꿔준다.

```sh
pick k51251 my father made this commit
pick v51241 my cat made this commit
drop e14124 not useful commit
pick g15155 initial commit
```

- 그럼 해당 commit의 내용이 자식 commit부터 싹 사라진다.
- 파일을 추가한 경우 파일도 사라진다.
- 다만 이모든 과정은 local에만 있는 commit일 떄 유효하다.
- rebase는 commit hash 값을 바꾸기 때문에 이미 올라간 commit에 대해선 하지말자.
  - 자기 branch안에서만 있는 경우는 괜찮지만, 타인에게 공유될 위험이 있기에 위험하다.



## <span style="color:#802548">_commit 정렬용 rebase -i_</span>
- 내 local main branch에 commit을 하고서 remote를 보았더니 1 commit behind였다.
  - 내가 PR로 다른 branch에서 commit을 github 상으로 넣었기 때문이었다.
- 그래서 본능적으로 pull origin main을 하게 됐다.
- 이 경우에 push를 해버리면 push는 되지만, linear history가 유지되지 않아 흐름을 알아보기 어렵다.
- 이럴 때 git rebase -i를 사용하면 추가된 local commit이 pull 받은 main 뒤에 붙게 된다.
- local commit은 어차피 remote에 push되지 않았기 때문에 rebase 해도 된다.
  - 단, 무조건 해당 commit만 rebase해야된다. 원격에서 pull받은 건 절대로 rebase하면 안 된다.

```sh
git rebase -i 
:wq
```



## <span style="color:#802548">_local에서 작업한 rebase 취소하기_</span>
- 위에처럼 rebase -i로 뭔가를 변경헀는데, 그게 실제론 remote에 있던 commit 이었다는 걸 알게 됐다.
- 그럼 rebase 했던 작업을 되돌려줘야 한다. 여태까지 나만 썼던 remote라면 상관없다.
- 하지만 모두가 공유하게 되는 branch에 들어가는 순간, 절대로 rebase되어선 안된다. 
  - commit history에 충돌이 생겨버린다.

- 원래의 commmit history의 commit value다.

```sh
16303e5 (HEAD -> feature/board) commit2
faf9dc2 commit1
72c04be commit
```

- commit이라는 이름이 마음에 안들어서 바꾼 뒤의 commit hash 값이다.

```sh
aa28634 (HEAD -> feature/board) commit2
33514fe commit1
ffcf8a9 d is left
```


- 서로 완전하게 다른 commit hash 값이니 충돌이 나게 된다.
- 따라서 reflog를 이용해 다시 되돌린다.
- 아래처럼 rebase의 start 지점이 있을 것이다. 해당 commit으로 되돌려야 한다.

```sh
aa28634 (HEAD -> feature/board) HEAD@{0}: rebase (finish): returning to refs/heads/feature/board
aa28634 (HEAD -> feature/board) HEAD@{1}: rebase (pick): commit2
33514fe HEAD@{2}: rebase (pick): commit1
ffcf8a9 HEAD@{3}: rebase (reword): d is left
72c04be HEAD@{4}: rebase: fast-forward
555313d HEAD@{5}: rebase (start): checkout 555313d
```

- 찾았다면 reset hard로 되돌린다. 
  - 참고로 아직 원격 repo에 push 하지 않았다는 가정이다.
  - 만약 push 했다면 revert로 rebase가 있기 전 까지 되돌려야 한다.
- reste hard로 찍은 commit hash값이 곧 head가 될 지점이다. 
- 한마디로 그 다음 찍은 commit은 무효화된다는 의미다.

```sh
git reflog
git reset --hard HEAD@{3}(ffcf8a9)
```



## <span style="color:#802548">_reflog_</span>
- git은 내가 쓴 명령어를 stack에 저장하는데, reflog를 통해 이를 확인할 수 있다.
- reflog에서는 움직임만을 HEAD로 잡아 알려주지, branch의 HEAD를 가리키지 않는다.
- 따라서 HEAD@{0}과 HEAD@{2}가 실제로 branch 내에서 같은 것을 ref할 수 있다.
  - 여기선 donkey branch의 HEAD다.
```
HEAD@{0}: checkout: moving from b151241251289512412 to donkey
HEAD@{1}: checkout: moving from donkey to b165125125fs1512515 //donkey HEAD에서 donkey 내 다른 commit으로 이동. detached HEAD
HEAD@{2}: commit: add haha
```

- 시간을 제약하여 reflog를 실행할 수도 있다.

```sh
git reflog show HEAD@{2.days.ago}
git reflog show HEAD@{1.month.ago}
git reflog show HEAD@{1.week.ago}
git reflog show master@{0} master@{yesterday}

# 2일 전까지 해당 branch에서 일어난 명령어들을 보여준다.
```

- 만약 rebase를 하다 잘못돼서 파일이 날라갔다면?
- 시간이 얼마 되지 않았다면 복구가 가능하다.
- .git 파일에서 해당 참조를 여전히 갖고 있기 때문이다.
- 다만 시간이 지나면 유효하지 않은 참조라서 git이 청소한다.

```sh
git reflog show
git reset --hard [commit hash]
```

## <span style="color:#802548">_특수 commit cherry-pick_</span>
- git cherry pick 했을 때 cherry pick 순서 지켜서 cherry pick 해야한다.
- git cherry pick은 주로 특정 branch에서 몇 개의 commit만 따오는 경우에 사용한다.
- 이렇게 따온 commit이 있게 되면, 해당 branch를 main에 push할 경우 이미 가져온 commit에 의해 conflict가 난다.
- 따라서 필요한 것만 따서 쓴 다음에 branch를 없애버리는 형태로 진행된다. 즉 다른 branch에 merge나 rebase하지 않는다.

```
cderf21
rtqwrr51
```

- 위와 같이 log 순서가 있다고 해보자.
- 그럼 아래와 같이 명령어를 써야 한다.
- git revert와는 반대다. 개념 상 당연하다.
  - revert는 최신 commit부터 되돌려야 하기 때문이고,
  - cherry pick은 이전 commit부터 쌓아야 하기 때문이다.

```sh
git cherry-pick rtqwrr51...cderf21 
git cherry-pick [oldest]...[latest]
```

- 거꾸로 하면 git conflict가 나게 된다.

```sh
git cherry-pick cderf21...rtqwrr51 
# git conflict
```

## <span style="color:#802548">_CLI branch diff_</span>
- PR을 하지 않고 master를 그냥 merge하고 배포한다면?
- 오류덩어리인 채로 배포하는 것이나 다름없다.
- 따라서 branch 간 diff를 통해 변화사항을 추적해야 한다.
- branch diff에 가장 좋은 방법은 github GUI를 사용하는 것이다.
  - 다시말해 그냥 github에서 Pull Request 올리라는 뜻이다.
- 하지만 그렇게 안하는 방법도 있긴 하다. 별로 추천하고 싶진 않지만 아래와 같은 이유로 사용했다.

```
자신은 PR을 승인할 권한이 없고, 회사 계정에만 PR을 승인할 권한이 있다고 해보자.
회사 github 계정이 모종의 이유로 로그인이 안 된다.
그런데 다행히 자기 계정에서 remote dev로 push하는 건 가능하다.
```

- 먼저 branch 간 diff를 해준다.
- 이것도 CLI로 일단 쓰지만, 매우 비추천이다. GUI로 하자.

```sh 
git diff [base branch] [tobe Branch]
git diff 
git diff origin/master master 
# 현재 local의 master와 remote의 master를 대비해줌. origin/master가 기준이 되고, local master branch를 비교해 추가되고 삭제된 것을 알려줌.
# git diff master origin/master는 반대. 따라서 이건 쓰면 안됨
```

- origin이 remote일 때 origin/master고, father가 remote라면 father/master로 해준다.

```sh
git diff father/master master
```

## <span style="color:#802548">_GUI branch diff_</span>

- git lens를 쓰면 훨씬 편하다.
- vscode의 도움을 받아 git diff를 실행할 수 있다.

```
https://yemsu.github.io/compare-two-git-branches-vscode/
```

- 아래 같은 과정으로 origin/master가 기준이 되어 local master branch를 비교한다.

```
git lens 설치
git lens inspect에서 SEARCH & COMPARE click
compare references click
처음 local의 master를 click
그 다음 origin/master를 click
```

- 혹시 잘못해서 안 보이면 명령 팔레트에서 compare 치면 나온다.

```sh
GitLens:Compare references
```

- 변화사항을 check해서 문제 없으면 넘기자

```sh
git switch main       # Switch to main
git merge feature       # Merge feature into main
git push origin main    # Push the updated main branch

# git push origin feature/recommend:main --> history가 동일해도 git pull 에러 뜨니 하지말자.
```

## <span style="color:#802548">_file diff_</span>

- 파일을 변경하고, staging에 올리지 않은 사항들을 보여주는 건 아래와 같다.
- 이 경우 새로운 파일을 등록하면 untraked라서 staging에 없어도 보이지는 않는다.

```sh
git diff
```

- 파일을 변경하고, staging에 올린 뒤에도 working과 staging 포함 변경사항을 보는 것은 아래와 같다.
- 새로운 파일을 등록하면 git add를 하고 git diff HEAD를 하면 변화사항을 포착할 수 있다.

```sh
git diff HEAD
```

- staging에 들어있는 변화사항만 보고싶다면 아래와 같다.

```sh
git diff --staged
```


- 특정 파일의 변화사항만 보고 싶다면 아래와 같다.

```sh
git diff HEAD [파일이름]
git diff --staged [파일이름]
git diff [파일이름]
```


- 특정 commit 간의 변화를 보고 싶다면 아래와 같다.

```sh
git diff [해쉬이름] [해쉬이름]
```

## <span style="color:#802548">_과거의 commit으로 프로젝트 되돌리기_</span>
- 특정 커밋을 본다고 표현했지만, 실제로는 특정 commit 버전으로 버전을 되돌리는 것이다.
- 그럼 detached head라고 뜨는데, 그건 원래 HEAD는 branch를 reference한다.
  - 따라서 branch의 맨 처음 commit을 따라간다.
  - 근데 여기선 특정 commit을 reference하기 때문에 branch에서 detached된 것이다.


```sh
git switch --detach <commit-hash>
```

- 특정 commit에서 다시 branch로 돌아가려면?


```sh
git switch - # 가장최근의 branch로 돌아감
```

- 만약 특정 commit까지의 이력만 가지고 local에 새로운 branch를 만들려면?
- 이 기능은 hotfix 실험하기 좋다.

```sh
git switch --detach <commit-hash>
git switch -c [branch]
```

## <span style="color:#802548">_git pull conflict 해결 법_</span>
- 기본적으로 git pull은 아래 명령어의 조합이다.
- 따라서 merge conflict가 날 수 있다는 의미다.

```sh
git fetch
git merge
```

- 헤결 대원칙은 아래와 같다.

```
1. 업무가 아닌 함수 단위 commit
2. 하루 3번 원격 저장소 동기화 - 출근 직후, 점심 먹고, 퇴근하기 1시간 전
```

- conflict가 나면 아래와 같다.

```sh
# <<<< ---> 이전에 있던 소스
# ===
# >>>> --> 새롭게 가져온 소스
```

- conflict는 아래와 같이 해결해준다.
- git diff는 CLI에서 하지 말고 꼭 GUI에서 수행하길 권장한다.
- unmerged의 경우, 수정하지 않고 그냥 git add . 해도 수정된 것처럼 인식된다. 
- 따라서 수정을 했으면 무조건 파일을 저장하고 git add를 해야한다.

```sh
git status 
## unmerged된 파일이 무엇인지 살펴본다.

git diff 
## git status로 확인한 문제되는 파일에서 어떤 부분이 문제인지 살펴보고 원하는 방향으로 정리한다.
## 이 명령어를 bash에서 쓰는 것은 추천되지 않는다. vscode에서 살펴보는게 훨씬 편하다

git add . 
## 수정한 사항을 staging area에 넣는다. 

git merge --continue 
## merge를 완료하는 commit을 남기면서 conflict를 해결한다.
```


## <span style="color:#802548">_merger가 아닌 rebase 방식의 pull_</span>

- main에서 feature branch로 commit을 받아오는 작업을 할 때, github의 PR로 진행했었다.
- 그런데 아래와 같은 문제가 떴다. 뜨는 순간 feature branch에서 main으로 자연스럽게 merge가 안된다.
  - 무조건 conflict 확정이다. 따라서 이런 경우가 없게 해야한다.

```
2 commit ahead, 2 commit behind 
```

- 원래 github에는 rebase and merge로 main에서 하위 branch로 source code를 가져오면 main과 동일한 commit이어도 commit hash가 다르다.
- 그래서 ahead와 behind가 같이 뜨게 되는 것이다.
- 저렇게 되면 새로운 commit을 넣고 push하려면 conflict가 나게 되는 게 일상이 된다.
  - 그런데 같은 파일 겹치는 게 있으면 그대로 conflict가 된다.
  - 무조건 main에서 하위 branch로 뿌릴 때는 up to date 상태가 되도록 해야한다. 
  - 만약에 main에 feature의 commit을 넣고 싶다면 말이다.
- 만약 동기화 작업을 하고 싶다면 아래와 같이 작업을 매 branch마다 거쳐줘야 한다.
  - 그렇게 되면 ahead, behind 같은 건 사라지고 up to date가 된다.
  - fetch로 가져오게 되면 commit hash 값이 다른데, 이를 rebase하게 되면 main의 commit hash값과 동일하게 된다.
    - 즉, rebase는 특성상 force push가 강제된다. 그러니 더더욱 조심해야된다.

```sh
git checkout [feature/branch]
git fetch origin #(새로운 소스코드 가져옴)
git rebase origin/main #(commit hash값을 main과 동기화시킴)
git push --force-with-lease
```

- 현재 branch의 commit을 dev에다가 통합시키고 싶을 때 아래와 같이 rebase를 활용한다.
- target branch에 source branch의 commit을 박치기한다. 당연히 history가 동일함을 전제로 한다.

```sh
git rebase dev branch_1
# git rebase <target> <source>
```


- Rebase 할 branch가 현재 checkout 된 branch 라면, source branch는 생략가능하다.

```sh
git switch branch_1
git rebase dev
git rebase <target>
 ```

## <span style="color:#802548">_자주 쓰는 rebase 시나리오_</span>

- rebase의 간단한 시나리오를 살펴보자.
- git pull 없이 rebase로만 진행되는 시나리오다.
  - git pull은 git fetch + git merge다.
  - upstrea의 dev에 넣는 것을 목적으로 한다.

```
fork를 딴다
fork repo를 clone한다
upstream을 설정한다.
upstream에서 최신 dev 코드를 받아온다.
최신화된 dev branch 에서 새로운 branch를 따서 작업한다
작업 및 Test가 완료되면 작업한 브랜치의 commit들을 local의 dev 기준으로 rebase한다.
원격 upstream dev 에서 변경점이 있다면, upstream dev를 기준으로 다시 rebase를 시켜 upstream과 sync를 맞춰준다
이후 push를 통해 commit hash 값을 동일하게 유지한다.
fork한 repo의 github에서 원본 repo로의 pull request를 날려준다
dev에서 받아들여지면 fork repo도 동기화한다.
```

- 위의 시나리오에 사용되는 명령어는 아래와 같다.

```sh
---------fork는 알아서---------
git clone -b dev <your_fork_repo_url>
git remote add upstream https://github.com/original-owner/repo.git
git fetch upstream
---------upstream 설정완료---------
git switch dev                # Switch to the dev branch
git pull upstream dev --rebase   # Ensure your local dev is up to date
git switch -c feat/test        # Create a new branch for your feature
---------upstream의 code 반영 완료-----------
git add .                      # Stage changes
git commit -m "Add new feature" # Commit changes
---------기능 추가 완료---------
git switch feat/test
git rebase dev                 # Move your commits on top of the latest dev
-------dev 기준으로 feat/test branch에 완료---------
git fetch upstream
git rebase upstream/dev        # Reapply your commits on top of the latest upstream/dev
------upstream dev에 추가된 코드 반영 완료------------
git push --force-with-lease origin feat/test
------commit hash 값 동일하게 유지 완료-----------
Github 에서 Pull Request 날리기 
받아들여져서 dev에 들어가면 fork repo를 동기화한다.
```

## <span style="color:#802548">_태그_</span>
- tag를 다는 이유는 중요 분기점을 구분하기 위해서다.
- 1차 버전, 2차버전, 3차버전 등의 구분을 commit에 tag를 달아서 할 수 있다.

```sh
git tag [tagname]
# 태그를 생성한다.

git tag -a [tagname] -m "[message]"
# 메타데이터를 포함한 태그를 생성한다. message가 필수다.

git show [tagname]
# -a 태그로 생성한 태그의 경우, 태그 생성 날짜, 태그 생성인 등의 메타데이터를 보여준다.

git tag  [tagname] [commit hash]
# 해당 commit에 tag를 달아준다.

git tag [tagname] [commit hash] -f 
# 이미 존재하는 tag를 다른 commit으로 이동시킨다.

git tag -l 
# tag를 달은 모든 목록을 보여줌.

git tag -l "17" 
# 정확히 17이라는 tag를 보여줌

git tag -l "*17*"
# 17이 들어간 태그를 보여줌

git tag [태그] [해쉬값] -m "message" : 
# git tag -l에는 message가 없지만 git show [tag이름]하면 부여한 message가 나옴.

git tag -d [태그이름] 
# 태그 삭제. d는 delete

git push origin [tagname]
# 해당 tag만 remote에 동기화

git push origin --tags :  
# 로컬저장소의 모든 태그를 원격저장소에도 동기화

git push origin --delete [태그이름] : 
# 특정 태그만 원격저장소와 로컬저장소 모두에서 삭제

git checkout [tag] 
# 해당 tag가 달린 commit으로 HEAD가 이동한다.

git diff [tag] [tag]
# commit간의 비교와 동일하게 작동
```
​

## <span style="color:#802548">_대용량파일 lfs 올리기_</span>
​- 혹시 npm 알집 파일 전체가 올라가서 lfs가 가동되는 경우는 npm 알집 파일은 전부 지우자.
- npm 알집들은 local에서 download하는 것이지, github에 올리는 파일이 아니다.
- npm은 dependency를 갖고 있는 config 파일만 github에 올려야 한다.
- 그러한 일이 없게 gitignore 파일도 추가해주자.
- 다만 lfs는 일정 이상 쓰면 유료다.

```sh
git init : 
# 맨 처음 git의 영역임을 선언.

git lfs install : 
# lfs를 설치

git lfs track [파일명] :
# lfs가 100MB가 넘는 파일을 추적하게 함. staging area에 올리기 전에 먼저해야됨.

git add . : 
# staging area에 올림

git commit -m"메시지" : 
# local 저장소에 확실히 저장

git remote add origin [url] : 
# 원격저장소 가져오기

git push origin master : 
# 원격저장소에 lfs가 포함된 파일까지 올리기
```

## <span style="color:#802548">_프로젝트 도중에 git ignore 만들어 적용_</span>
- git ignore 파일 같은 경우, IDE를 통해 Gradle 혹은 Maven project를 만들면 전부 이미 포함되어 있다.
- 혹시 없는 경우에는 gitignore를 처음에 넣고 시작하는 게 좋다.
- 하지만 까먹은 경우 아래와 같이 적용한다.

```sh
git rm -r --cached . 
## 원격저장소에 있는 파일들을 모두 삭제한다. 로컬저장소는 유지된다.

vim .gitignore 
## 무시하고자 하는 파일들을 써 준 뒤 :wq로 저장한다.

git add . 
## 원격저장소에 아무것도 없으므로 전부 다시 넣는다.

git commit -m"git ignore add" 
## gitignore를 추가했다는 기록을 남긴다.

git push origin master 
## 다시 원격저장소에 올린다. 
```

## <span style="color:#802548">_프로젝트 도중에 tracking된 파일을 git ignore에 추가하기_</span>

- 아래와 같이 하면 gitignore에 무시하려는 파일을 추가할 수 있다.
- 대신에 아래와 같이 하게 되면 git pull을 받는 순간 파일이 사라지기 때문에 자료를 받아야 한다.

```sh
.gitignore에 무시를 원하는 파일 추가하기
git rm --cached application.properties
git add .gitignore 
git commit -m "ignoring file"
git push
```



## <span style="color:#802548">_push 시 permission denied 오류_</span> 
- remote에 push가 안되는 경우가 있다.
- 안된다기보다는 2021년 이후론 credential manager 버전이 사라져서 password 방식을 지원하지 않는다.
- 따라서 ssh 방식으로 진행해야 하는데, 계속 password authentication으로 작동하는 것이다.

- 처음에는  SSH-ADD 명령어가 안들어서 SSH-ADD 오류인줄 알았다.
- 하지만 원인은 remote가 HTTPS였던 것이었다. HTTPS는 password authentication 방식 고정이다.
- SSH 공용키 등을 다 등록하고, remote를 지우고 SSH url로 다시 remote를 설정하고 push를 하면 잘 작동한다.


## <span style="color:#802548">_github issue 활용하기_</span>

```
repo 만들기
invite peopl
collaborators - add people
projects 클릭 - link projects - create new projects - board로 생성
add feature를 Todo나 In Progress Done에 추가
feature 클릭해서 add assign - convert to issue
그럼 자동으로 issue가 열리는데 거기서 create a branch 누르면 remote에서 만든 branch가 생성. 
local에서 해당 branch 활성화 하려면 아래 command 입력
git fetch origin
git switch [feature branch] //그럼 알아서 upstream이 origin/featrue branch가 되어 track됨.
그리고 pull request하면 됨.
```



## <span style="color:#802548">_git commit 전부 초기화하면서 폴더 내용 그대로 가져가기_</span>
- React를 js에서 ts로 바꾸는데 이전 commit은 전부 없애고 새로 시작하려고 한다. 
- 그러면서 동시에 작업했던 내용은 가져가고 싶다면 아래와 같은 방법을 활용할 수 있다. 

```sh
rm -rf .git
git init
git add .
git commit "initial commit"
git remote add origin "url"
git push origin master
```

- 그럼 첫 legacy 프로젝트를 처음 commit부터 시작할 수 있다.



## <span style="color:#802548">_git folder명 바꾸기_</span>
- 그냥 폴더명만 바꾸고서 add commit하면 변화가 관찰되지 않는다.
- 그렇게 git mv를 활용해야 한다.
- 우선 그냥 경로에서 진행하면 에러가 난다.

```
fatal: bad source, source=DTO, destination=foldessd
```

- 이름을 바꿀 경로까지 이동해주자.

```
cd src/main/java/org/scit/project/user
```

- 그러고 나서 폴더를 바꾸려고 하면 이름이 제대로 바뀌지 않는다.

```js
git mv DTO foldessd
//fatal: renaming 'src/main/java/org/scit/project/user/DTO' failed: Invalid argument
```

- 이동 하고서도 그냥 dto로 바꾸면 안되고 temp_folder 같은 식으로 바꿔야됨
- git config core.ignorecase false 로 놓지 말기. 다른 사람한테 다 영향 갈 수 있으니 쓰지 말자

```
git mv DTO temp_folder
git mv temp_folder dto
```
- 그러고 나서 git status를 하면 바뀐게 보인다.
- 직접 git add로 git bash에서 추가해주자.


## <span style="color:#802548">_vscode git 변화사항 추적이 안될 떄_</span>
- 혹시 gradle build에서 error가 나면 git 등에서도 변화 사항이 반영되지 않는다.
- output에 보면 error가 나는 지 안 나는 지 확인 가능하다.
- 아래처럼 나는 에러가 나고 있었다.

```
Could not execute build using connection to Gradle distribution 'https://services.gradle.org/distributions/gradle-8.12.1-bin.zip'.
C:\Users\user\OneDrive\바탕 화면\tanomuzoko\src\main\java\org\scit\project\board_heart\entity\BoardHeartEntity.java:46: warning: @Builder will ignore the initializing expression entirely. If you want the initializing expression to ...
```


- 프로젝트 compile 시에 빨간줄이 없는데 위와 같은 에러가 뜬 이유는, 폴더명 변경때문이다..
- 따라서 그냥 gradle을 전부 clean 시킨다. 해당 명령어는 git bash에서 그냥 실행하면 된다.

```sh
./gradlew clean build
```




## <span style="color:#802548">_git merge commit message에 따른 차이_</span>

- github에서 PR을 올리게 되는 경우의 commit 메세지는 아래와 같다.

```

```

- github에서 PR을 올리려는데, conflict가 너무 많아서 resolve를 GUI의 도움을 받아서 해야한다면, GIT BASH에서 진행하게 된다.
- git merge로 가져다 박을 탠데, 그 때 commit message는 아래와 같다.

```

```

- main에 올라온 PR을 받아서 git pull을 하게 되면 local에서 message는 아래와 같다.

```

```

- remote의 branch와 자기 local branch가 다른 경우, git pull로 올려 받은 경우에는 아래와 같이 된다.
  - 다른 사람이 대신 자기 local branch에서 무언가를 해준 것을 받을 때
  - 폴더명은 소문자 원칙이라 대문자 폴더를 소문자로 바꿨지만, git은 case-insensitive라서 인지가 되지 못하고 github에 올라간 경우

```
Merge remote-tracking branch 'origin/develop' into develop
```


## <span style="color:#802548">_github 상 PR message가 어떻게 나오나_</span>

- 하나만 commit 한 branch를 squash merge 하면 그냥 commit의 message가 들어가게 된다.

```
DTO -> dto로 폴더명 변경 적용용 (#23)
```

- 반면에 복수개의 commit을 한 branch는 squash merge를 하면 commit의 message가 아래처럼 나온다.

```
Feature/login join (#22)
```


## <span style="color:#802548">_vscode git source control_</span>

- U는 untrackted다. 새로 추가된 파일이란 의미다.
- M은 modified다.
- D는 deleted다.
- R은 rename이다.


## <span style="color:#802548">_github 상 PR message 규칙_</span>
- title은 아래처럼 작성한다.

```
Feature: User profile page (Merge from `feature/profile` to `dev`) (#34)
```

- 실제 내용은 commit을 포함한다.

```
This PR merges `feature/profile` into the `dev` branch.

- Created user profile page UI.
- Integrated with backend API to fetch user data.
- Added routing for profile navigation.

Closes #34
```


