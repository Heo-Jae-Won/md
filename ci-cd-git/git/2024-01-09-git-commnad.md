## <span style="color:#802548">_git pull request_</span>
pull request하는 방법은 아래에 잘 서술되어 있다.  

https://wayhome25.github.io/git/2017/07/08/git-first-pull-request-story/


- 만약 github clone 시 fatal: repostiroy not found가 뜰 경우

- git clone https://github.com/id/reponame.git와 같이 했는데 위와 같은 오류가 날 수 있다.
- 그 경우 git clone https://userid@github.com/id/reponame.git와 같이 앞에 username을 붙여준다
- 그리고 나면 해당 유저로 로그인하여 증명하라는 표시가 뜬다. 로그인하여 증명하면 clone이 가능하다.
- 기본적으로 해당 오류가 나는 것은, 권한을 얻지 못했기 때문이다. 
- private repo에 접근하려는데 권한이 없는 경우 자주 발생한다. 따라서 해당 아이디로 로그인해서 credentials manager를 얻는 것이다.
- 만약 git clone을 할 게 master가 아니라 dev라면 아래와 같이 dev를 clone한다.
```sh
git clone -b(--branch) <branchname> <remote-repo-url>
```
- 혹시 까먹고 master로 가져왔어도 상관없다.
- 보통 github clone으로 가져온경우, 보통 master밖에 없을 것이다.

- 그럴 때 git branch --all
- git checkout [원하는 branch]로 원하는 branch를 가져온다.

- 혹시 도중에 다른 사람이 branch를 remote에 추가됐다면?
- git remote update로 remote에 추가된 branch를 내 git file에 인식시킨다.
- 그 뒤 git checkout [원하는 branch]로 하면 가져올 수 있다.


## <span style="color:#802548">_1. branch 관련 명령어_</span>
```sh
git switch -c [branch이름] : 
## git branch test + git switch test : branch를 만들고 바로 이동.
git checkout [branch이름] : 
## working dir의 소스코드가 해당 commit의 소스코드로 변화한다.
git push origin --delete [branch이름] : 
## 로컬저장소와 원격저장소 모두에서 branch 삭제.
git branch -d [branch이름] : 
## 로컬저장소에서 branch를 삭제.
git branch --move [바꿀branch이름] [바뀐branch이름] : 
## branch 이름을 바꿀 수 있는 기능이다. 
```

​

## <span style="color:#802548">_2. staging area 관련 명령어_</span>
```sh
git add -u : 
## 수정하고 삭제한 것들만 반영하며, 새로 추가된 파일은 staging area에 반영되지 않는다. 
```

```sh
git add . : 
## 수정, 삭제, 추가된 파일들을 전부 반영한다. git ignore에 있는 파일은 제외된다. 만약 git add를 하고 싶지 않다면 아래에서 볼 git stash를 이용한다. 
```

```sh
git restore --staged . : 
## git add한 것을 모두 staging area에서 제외시킨다.
```

```sh
git add [file이름] 
git commit --amend -m "Add new File" : 
## commit을 했는데 빼먹은 파일이 있다면 staging area에 넣고 다시 commit함. 메시지를 바꿀 생각이 없다면 --amend까지만 하면 됨. 
```

​

## <span style="color:#802548">_3. commit이력을 혼자 작업 시 로컬저장소에서 삭제하는 명령어_</span>
```sh
git reset --mixed : 
## 커밋이력을 없애면서 git add한 것들을 모두 없앤다. 소스코드는 유지된다. 

git reset --soft : 
## 커밋이력을 없애면서 여태까지 git add한 것을 보존하면서 소스코드는 유지된다. 
## 커밋 이후에 뭔가 더 추가해야한다면 자주 쓴다. git reset --soft + git add . + git commit -m""의 형태다. 
# 위처럼 git add [file이름] + git commit --amend -m "commit"과 같다.

git reset --hard : 
## working dir과 staging area 모두에서 지운다
```
​

## <span style="color:#802548">_4. commit이력을 협업할 때 로컬저장소에서 삭제하는 명령어_</span>

```sh
git revert --no-commit [해쉬코드]...[해쉬코드] : 
## 여러 개의 commit이력을 없애고자 할 때 전부에 대해 revert commit을 남기지 않고 1개의 commit만 남긴다.

git revert --no-commit HEAD~3
## 여러개를 revert 해도 commit은 1개만 남는다. 각각의 revert commit을 남기지 않는다.

git push origin master
```
​

## <span style="color:#802548">_5. 혼자 작업 시 원격저장소에서 commit이력을 삭제하는 법_</span>
```sh
git reset --hard

git push origin master -f
```
​

## <span style="color:#802548">_6. git pull 시에 conflict 해결 법_</span>
- 대원칙은 아래와 같다.
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
```sh
git status : 
## unmerged된 파일이 무엇인지 살펴본다.

git diff : 
## git status로 확인한 문제되는 파일에서 어떤 부분이 문제인지 살펴보고 원하는 방향으로 정리한다.
## 이 명령어를 bash에서 쓰는 것은 추천되지 않는다. vscode에서 살펴보는게 훨씬 편하다

git add . : 
## 수정한 사항을 staging area에 넣는다. unmerged의 경우, 수정하지 않고 그냥 git add . 해도 수정된 것처럼 인식된다. 따라서 수정을 했으면 무조건 파일을 저장하고 git add를 해야한다.

git merge --continue : 
## merge를 완료하는 commit을 남기면서 conflict를 해결한다.
```
​

## <span style="color:#802548">_7. 원격저장소 설정_</span>
- remote는 보통 이름을 origin으로 정한다.

```sh
git remote add [저장소 alias] [url] :
## 원격저장소를 설정한다. 거의 git remote add origin https://~~~ 형태

git remote -v : 
## 설정된 원격저장소와 저장소이름 정보 출력

git remote set-url [저장소 alias] [url] : 
## 기존 원격저장소를 새로운 원격저장소로 바꿈

git remote remove [저장소 alias] : 
## 원격저장소와 연결 끊으며, 저장소 alias도 사라진다.
```
​

## <span style="color:#802548">_8. 작업파일 임시 저장_</span>
- 변화를 가지고 git switch branch하면 되는 경우도 있다.
  - git add를 안해도 되는 경우는, conflict가 없는 경우다.
  - 이동하려는 branch가 fast-forward가 가능하다면 git add를 하지 않아도 branch switch가 가능하다.

- 만약 conflict가 일어날 것이 분명하다면, git switch가 불가능하다.
- 그럴 때 git stash를 사용해 local changes를 저장한다.

- 버그를 해결할 때, 여러 시도를 할 때도 git stash를 여러번하여 실패한 버전과 실패했던 이유를 떠올릴 수 있다.
- 개인 공부할 때 도움이 될 것이라 생각된다.
- 다만 버그를 해결하면 git stash clear를 꼭 해줘야 한다.
  - 그렇지 않으면 commit history graph에 stash도 남아있어 굉장히 지저분해진다.


```sh
git stash : 
## 작업 파일 임시 저장

git switch -C feature/insert : 
## insert branch를 만들면서 이동.
git stash apply : 

## feature/insert branch에 임시로 저장해둔 작업 파일을 꺼냄. 

git stash branch [branch이름] : 
## git switch -C [branch이름] + git stash apply + git stash drop을 한꺼번에 한다. 단 stash에 내용이 있어야만 하기 때문에 git stash + git stash branch [branch이름] 식으로 사용한다.

git stash drop stash@{0} : 
## 0번째 index stash 삭제

git stash list : 
## 임시저장 파일 목록 확인

git stash clear : 
## 전체 stash를 다 비운다
```
​

## <span style="color:#802548">_9. 프로젝트 도중에 git ignore 적용_</span>

```sh
git rm -r --cached . : 
## 원격저장소에 있는 파일들을 모두 삭제한다. 로컬저장소는 유지된다.

vim .gitignore : 
## 무시하고자 하는 파일들을 써 준 뒤 :wq로 저장한다.

git add . : 
## 원격저장소에 아무것도 없으므로 전부 다시 넣는다.

git commit -m"git ignore add" : 
## gitignore를 추가했다는 기록을 남긴다.

git push origin master : 
## 다시 원격저장소에 올린다. 
```
​

## <span style="color:#802548">_10. commmit 이력 보기_</span>
- git commit 이력 보는 것은 아래가 기본이다.
- 하지만 alias를 설정해주는 게 좋다.
- https://www.durdn.com/blog/2012/11/22/must-have-git-aliases-advanced-examples/ 를 참고하자.


```sh
git log --oneline : 
# commit 로그를 짧게 중요한 정보만 요약해 출력함.
```
```sh
git log -p -n : 
# commit 로그를 바뀐 내용과 함께 n개만큼 보여준다.

git log --since="2023-04-02" : 
# 해당 날짜 이후의 commit 이력만 출력한다. 

git log --author=username : 
# 해당 username이 남긴 commit 이력을 확인한다.

git log --grep="hotfix" : 
# hotfix라는 commit 메시지를 출력한다.

git log -S "change" : 
# change라는 소스코드가 들어간 commit을 출력한다.
```

​

https://www.atlassian.com/git/tutorials/git-log

 
## <span style="color:#802548">_11. 환경설정_</span>

```sh
git config --list : 
# 설정해둔 변수들을 확인한다.

git config --global user.name "유저이름" : 
# 해당 유저이름으로 commit이 남게 설정한다.

git config --global user.email "이메일" : 
# 해당 이메일로 commit이 남게 설정한다.

git config --unset 속성(user.name) : 
# 해당 속성의 값을 삭제한다.\

git config --global init.defaultbranch main :
# default branch의 이름을 master가 아닌 main으로 한다
```
​

​

## <span style="color:#802548">_12. 태그_</span>
​
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

git tag -l : 
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

## <span style="color:#802548">_13. 대용량파일 lfs 올리는 순서_</span>
​

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
​

## <span style="color:#802548">_14. 별칭 설정하기_</span>
​
```sh
vim ~/.gitconfig: 
# git의 설정을 연다. 아래와 같이 alias를 선언하고 쭉 써주면 된다. 참고로 저기 pn은 먹히지 않는다. gitconfig는 git에서 인식 가능한 command만을 alias로 설정할 수 있다. pnpm은 또 다른 system이라서 인식되지 않는다.
```

- git config --list로 쳐보면 설정한 email, name, alias가 잘 들어온 것을 확인할 수 있다.


​

## <span style="color:#802548">_15. 로컬에서 이전 커밋 메시지 수정하기_</span>
```sh
git rebase -i HEAD~n : 
# 현재 commit을 포함해 3 commit을 내려가 거기를 HEAD로 바꿈
pick을 reword로 변경하고 :wq 
commit message를 변경하고다시 :wq
```
 

## <span style="color:#802548">_16. git commit 전부 초기화하면서 폴더 내용 그대로 가져가기_</span>
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

- 그럼 아래와 같은 형태로 전부 보존하며 새롭게 conversion 할 수 있다. 


## <span style="color:#802548">_17. local change 지우기_</span>
- 아직 staging에 올라오지 않은 것은 아래와 같이 지울 수 있다.


```sh
git checkout HEAD [filename]
git checkout -- [filename]
git restore [filename]
```

- 한꺼번에 다 지우고 싶다면?


```sh
git checkout -- .
git restore .
```

- 만약 파일 내용을 전전 commit으로 돌리고 싶다면? 그러면서 HEAD는 유지하고 싶다면?


```sh
git restore --source HEAD~2 [파일이름]
git restore --source [해쉬이름] [파일이름]
```

- 다시 HEAD로 복구하고 싶다면?


```sh
git restore [파일이름]
```


## <span style="color:#802548">_18. 체리픽_</span>
- git cherry pick 했을 때 cherry pick 순서 지켜서 cherry pick 해야한다.

```
cderf21

rtqwrr51

```
- 위와 같이 log 순서가 있다고 해보자.
- 그럼 아래와 같이 명령어를 써야 한다.

```sh
git cherry-pick rtqwrr51...cderf21 
```
- 거꾸로 하면 git conflict가 나게 된다.

```sh
git cherry-pick cderf21...rtqwrr51 
# git conflict
```

## <span style="color:#802548">_19. branch diff_</span>
- PR을 하지 않고 master를 그냥 merge하고 배포한다면?
- branch 간 diff를 통해 변화사항을 추적해야 한다.

```sh 
git diff [base branch] [tobe Branch]
git diff 
git diff origin/master master 
# 현재 local의 master와 remote의 master를 대비해줌. origin/master가 기준이 되고, local master branch를 비교해 추가되고 삭제된 것을 알려줌.
# git diff master origin/master는 반대. 따라서 이건 쓰면 안됨
```
- origin이 remote일 때 origin/master고, father가 remote라면 father/master로 해줌.
```sh
git diff father/master master
```

- git lens를 쓰면 훨씬 편함.
- vscode의 도움을 받아 git diff를 실행할 수 있음.

https://yemsu.github.io/compare-two-git-branches-vscode/
- 아래 같은 과정으로 origin/master가 기준이 되어 local master branch를 비교
```
git lens 설치
SEARCH & COMPARE
compare references
처음 local의 master를 클릭
그 다음 origin/master를 클릭
```

- 파일을 변경하고, staging에 올리지 않은 사항들을 보여주는 건 아래와 같다.
- 이 경우 새로운 파일을 등록하면 untraked라서 staging에 없어도 보이지는 않는다.

```
git diff
```

- 파일을 변경하고, staging에 올린 뒤에도 working과 staging 포함 변경사항을 보는 것은 아래와 같다.
- 새로운 파일을 등록하면 git add를 하고 git diff HEAD를 하면 변화사항을 포착할 수 있다.

```
git diff HEAD
```

- staging에 들어있는 변화사항만 보고싶다면 아래와 같다.


```
git diff --staged
```


- 특정 파일의 변화사항만 보고 싶다면 아래와 같다.


```
git diff HEAD [파일이름]
git diff --staged [파일이름]
git diff [파일이름]
```


- 특정 commit 간의 변화를 보고 싶다면 아래와 같다.


```
git diff [해쉬이름] [해쉬이름]
```

## <span style="color:#802548">_20. 해당 commit 보기_</span>
- 특정 커밋으로 돌아가기
- 그럼 detached head라고 뜨는데, 그건 원래 HEAD는 branch를 reference한다.
  - 따라서 branch의 맨 처음 commit을 따라간다.
  - 근데 여기선 특정 commit을 reference하기 때문에 branch에서 detached된 것이다.


```sh
git checkout [commit hash]
```

- 특정 commit에서 다시 branch로 돌아가려면?


```sh
git switch [branch]
git switch - (가장최근의 branch로 돌아감)
```


- 만약 특정 commit까지의 이력만 가지고 새로운 branch를 만들려면?


```sh
git checkout [commit hash]
git switch -c [branch]
```


- remote branch 따라가기


```sh
git branch -r # 원격 branch 뭐있나 보기
git switch [원격branch의 이름] # 이러면 저절로 remote의 branch와 연결되어 추적가능. 옛날버전은 git checkout --track origin/master. 쓰지말자.
```

## <span style="color:#802548">_21. rebase 활용법_</span>

- pull request를 하려는데 conflict가 나면 PR이 불가능하다.
  - 그럴 때는 view command line을 클릭해 어떻게 하는 지 보고 conflict를 해결해주면 된다.
  - 단, 그 경우 merge commit을 남기면 자동으로 PR이 닫힌다. conflict가 나면 PR이 불가능하다고 보면 되겠다.


- branch 보호 규칙도 설정가능하다.
  - 특히 master, main은 PR 필수로 섫정하는 경우가 많다.
  - repo - settings - rules에서 설정한다. 

- rebase를 하려면 어떻게 하냐?
  - master를 기준으로 다시 feature branch의 history를 재정비하라는 명령
  - feature의 commit은 master의 최신 commit 뒤로 붙게 된다.

  
```sh
git pull origin master
git switch [feature branch]
git rebase master
```

- 이렇게 git rebase를 한 다음에 PR을 올려주면 된다.


- git rebase를 하면 안되는 경우도 있다.
- 이미 remote에 push한 commit들이다.
- 같은 이유로 master branch에서 rebase하면 절대 안된다.
- feature branch 또한 remote에 push했고, 누군가 그 branch를 따서 작업을 진행한다면?
  - 그 때는 rebase하면 안 된다.


## <span style="color:#802548">_21. rebase -i_</span>

- github 쓰는 법.
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

- commit message 바꾸기도 rebase로 가능하다. 
- 하지만 remote에 안올렸을 때만 써야 한다.


```sh
git rebase -i HEAD~4
```

- 그럼 오래된 commit부터 위로 오게 vim이 열린다.
- 그 떄 pick을 reword로 바꿔준다.
- 그럼 커밋 메시지만 바꾸게 된다.
- 커밋 내용은 바뀌지 않지만, 커밋 해시는 새로 생긴다.
- rebase한 commit 이후의 commit들도 부모 commit의 해쉬가 바꼈으므로 같이 커밋 해쉬가 바뀐다.


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


- commit을 아예 삭제하는 것도 가능하다.


```sh
k51251 my father made this commit
v51241 my cat made this commit
e14124 not useful commit
g15155 initial commit
```

- 똑같이 현재 branch를 rebase 해준다.


```sh
git rebase -i HEAD~2
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


## <span style="color:#802548">_22. reflog_</span>
- reflog로 최근에 사용한 모든 명령어를 볼 수 있다.
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


- 버전의 숫자는 의미가 다르다.
- 1.0.0이 처음이다.
- 1.0.1로 가면 버그 패치다.
- 1.1.0으로 가면 신기능이 생긴 경우다.
- 2.0.0으로 가면 완전히 새롭게 개편한 경우다. 기능이 삭제되고, 메뉴가 완전 새롭게 바뀐다.


- git은 key와 value를 통해 저장한다.
- key는 SHA-1 방식으로 해싱된 hash 값(40자리 16진수 값)이고, value는 일종의 파일이다.

- 파일을 복원하려고 할 때, git reflog 외에도 아래와 같이 쓸 수 있다.


```sh
git cat-file -p [commit hash]
git cat-file -p [tree hash]
git cat -file -p [src tree hash]
git cat-file -p [원하는폴더 tree hash]
git cat-file -p [원하는 파일 blob hash] > [원하는 텍스트 이름]
```


- git이 저장하는 종류로는 blob, tree, commit이 있다.
- blob은 파일의 내용만을 포함한다.파일의 이름이 저장되지는 않는다.
- 파일의 이름은 tree가 저장한다. 정확히는 디렉터리의 구조를 저장한다.
- 큰 디렉토리를 하나의 tree라고 하면, 그 tree 안에 하부 파일의 tree와 blob이 같이 있는 구조다.

```
tree
tree blob tree blob
```


- tree를 보고 싶다면 아래와 같이 쓸 수 있다.
- 아래는 master tree 기준이다.

```sh
git cat-file -p master^{tree}
```

- 마지막으로 commit이 있다.
- commit은 tree, parent, author, commiter, message를 가진다.
- tree는 폴더 구조를 스냅샷으로 담는다.
- tree 안에 해당 프로젝트의 tree와 blob이 있기 때문에 해당 프로젝트를 전부 복원할 수 있다.
- 파일의 내용이 바뀌지 않으면 해당 blob은 계속 똑같은 hash값을 유지한다.
```sh
git cat-file -p [commit hash]

tree ....
parent ....
authro ...
committer...
message ...
```

- 파일의 내용이 바뀌면 새로운 blob이 형성된다. 
- 따라서 tree 안에서 새로운 blob으로 바뀌게 되고, 그게 스냅샷으로 찍힌다.
- 입력값이 아무리 바뀌어도 40자리의 commit hash를 생성한다. 따라서 파일로 저장하지 않고, 40자리 commit hash로 가지고 있다가 필요하면 복호화해서 파일 형태로 펼치는 것이다.
- 파일 형태로 펼칠 때 파일 이름은 tree가 알려주게 된다.


## <span style="color:#802548">_23. git object_</span> 
- git은 C언어로 내부에서 이뤄져서 file, folder, commit을 object로 관리한다.
- 해당 object들은 모두 SHA-1 방식으로 해싱 암호화가 이뤄진다.
- 향후 SHA-256 등 암호화 수준을 올릴 계획은 있다고 한다.
- SHA-1 해싱 암호화의 이점은 아래와 같다.
- 
```
각 파일마다 고유의 해쉬를 가진다. 따라서 유일성이 보장된다. 변화사항마다도 고유의 해쉬를 가지기 때문에 버전관리가 가능하다.
해쉬기반으로 검색을 빠르게 진행할 수 있다.
```
- file은 C언어의 blob이다. file을 blob으로 저장하면 좋은 장점은 아래와 같다.

```
변화사항이 없으면 새로운 blob을 만들지 않는다.
파일이었다면, 파일째로 다시 저장해야해서 저장소 용량을 효율적으로 활용할 수 없다.
```

- tree는 C언어의 struct다. 
- 대략 아래와 같은 구조라고 한다.

```C
struct tree_entry {
    mode_t mode;  // File mode (permissions)
    char name[256];  // File or directory name (assuming a maximum length of 255 characters)
    unsigned char sha1[20];  // SHA-1 hash of the blob or subtree
};

struct tree {
    int entry_count;  // Number of entries in the tree
    struct tree_entry entries[MAX_ENTRIES];  // Array of tree entries
    unsigned char sha1[20];  // SHA-1 hash of the tree object
};
```

- commit도 C언어의 struct다.
- 대략 아래와 같은 구조를 지니고 있다.
```C
struct commit {
    unsigned char tree[20];  // SHA-1 hash of the tree object representing the project's directory structure
    unsigned char parents[MAX_PARENTS][20];  // Array of SHA-1 hashes of parent commits (for merge commits)
    const char *author_name;
    const char *author_email;
    unsigned long author_timestamp;  // Timestamp of the author's commit
    const char *committer_name;
    const char *committer_email;
    unsigned long committer_timestamp;  // Timestamp of the committer's commit
    const char *commit_message;
    unsigned char sha1[20];  // SHA-1 hash of the commit object
};
```

- tree는 blob을 배열로 갖고 있고, commit은 tree를 배열로 갖고 있다.
- 이와 같이 물고물어지는 계층구조를 갖고 있으며, 주소값도 담고 있기 때문에 branching하기가 매우 쉽다는 장점이 있다.



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

## <span style="color:#802548">_git permission denied 오류_</span> 
- 그 다음으로 clone은 언제든 되지만, remote에 push가 안되는 경우가 있다.
- 안된다기보다는 2021년 이후론 credential manager 버전이 사라져서 password 방식을 지원하지 않는다.
- 따라서 ssh 방식으로 진행해야 하는데, 계속 password authentication으로 작동하는 것이다.

- 처음에는  SSH-ADD 명령어가 안들어서 SSH-ADD 오류인줄 알았다.
- 하지만 원인은 remote가 HTTPS였던 것이었다. HTTPS는 password authentication 방식 고정이다.
- SSH 공용키 등을 다 등록하고, remote를 지우고 SSH url로 다시 remote를 설정하고 push를 하면 잘 작동한다.

