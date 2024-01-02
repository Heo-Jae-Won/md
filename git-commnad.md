## <span style="color:#802548">_git pull request_</span>
pull request하는 방법은 아래에 잘 서술되어 있다.  

https://wayhome25.github.io/git/2017/07/08/git-first-pull-request-story/


- 만약 github clone 시 fatal: repostiroy not found가 뜰 경우

- git clone https://github.com/openobjectnet/open-dashboard-front.git와 같이 했는데 위와 같은 오류가 날 수 있다.
- 그 경우 git clone https://openobjectnet@github.com/openobjectnet/open-dashboard-front.git와 같이 앞에 username을 붙여준다
- 그리고나면 해당 유저로 로그인하여 증명하라는 표시가 뜬다. 로그인하여 증명하면 clone이 가능하다.
- 기본적으로 해당 오류가 나는 것은, 권한을 얻지 못했기 때문이다. private repo에 접근하려는데 권한이 없는 경우 자주 발생한다. 따라서 해당 아이디로 로그인해서 credentials manager를 얻는 것이다.
- 만약 git clone을 할 게 master가 아니라 dev라면 아래와 같이 dev를 clone한다.
```sh
git clone -b(--branch) <branchname> <remote-repo-url>
```
- github clone으로 가져온경우, 보통 master밖에 없을 것이다.

- 그럴 때 git branch --all
- git checkout [원하는 branch]로 원하는 branch를 가져온다


## <span style="color:#802548">_1. branch 관련 명령어_</span>
```sh
git switch -C [branch이름] : 
## git branch test + git switch test : branch를 만들고 바로 이동.
```
```sh
git checkout [branch이름] : 
## working dir의 소스코드가 해당 commit의 소스코드로 변화한다.
```
```sh
git push origin --delete [branch이름] : 
## 로컬저장소와 원격저장소 모두에서 branch 삭제.
```
```sh
git branch -d [해당 branch이름] : 
## 로컬저장소에서 branch를 삭제.
```sh
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
git add [new file] & git commit --amend -m"Add new File" : 
## commit을 했는데 빼먹은 파일이 있다면 staging areadp 넣고 다시 commit함. 메시지를 바꿀 생각이 없다면 --amend까지만 하면 됨. 
```

​

## <span style="color:#802548">_3. commit이력을 혼자 작업 시 로컬저장소에서 삭제하는 명령어_</span>
```sh
git reset --mixed : 
## 커밋이력을 없애면서 git add한 것들을 모두 없앤다. 소스코드는 유지된다. 
```
```sh
git reset --soft : 
## 커밋이력을 없애면서 여태까지 git add한 것을 보존하면서 소스코드는 유지된다. 커밋 이후에 뭔가 더 추가해야한다면 자주 쓴다. git reset --soft + git add . + git commit -m""의 형태다. 
```

```sh
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
```gi
​

## <span style="color:#802548">_5. 혼자 작업 시 원격저장소에서 commit이력을 삭제하는 법_</span>
```sh
git reset --hard

git push origin master -f
```
​

## <span style="color:#802548">_6. git pull 시에 conflict 해결 법_</span>

```sh
git status : 
## unmerged된 파일이 무엇인지 살펴본다.
```
```sh
git diff : 
## git status로 확인한 문제되는 파일에서 어떤 부분이 문제인지 살펴보고 원하는 방향으로 정리한다.
```

```sh
git add . : 
## 수정한 사항을 staging area에 넣는다. unmerged의 경우, 수정하지 않고 그냥 git add . 해도 수정된 것처럼 인식된다. 따라서 수정을 했으면 무조건 파일을 저장하고 git add를 해야한다.
```

```sh
git merge --continue : 
## merge를 완료하는 commit을 남기면서 conflict를 해결한다.
```
​

## <span style="color:#802548">_7. 원격저장소 설정_</span>
```sh
git remote add [저장소 alias] [url] :
## 원격저장소를 설정한다.
```

```sh
git remote -v : 
## 설정된 원격저장소와 저장소이름 정보 출력
```

```sh
git remote set-url [저장소 alias] [url] : 
## 기존 원격저장소를 새로운 원격저장소로 바꿈
```

```sh
git remote remove [저장소 alias] : 
## 원격저장소와 연결 끊으며, 저장소 alias도 사라진다.
```
​

## <span style="color:#802548">_8. 작업파일 임시 저장_</span>
```sh
git stash : 
## 작업 파일 임시 저장
```

```sh
git switch -C feature/insert : 
## insert branch를 만들면서 이동.
```

```sh
git stash apply : 
## feature/insert branch에 임시로 저장해둔 작업 파일을 꺼냄. 
```

```sh
git stash branch [branch이름] : 
## git switch -C [branch이름] + git stash apply + git stash drop을 한꺼번에 한다. 단 stash에 내용이 있어야만 하기 때문에 git stash + git stash branch [branch이름] 식으로 사용한다.

```sh
git stash drop stash@{0} : 
## 0번째 index stash 삭제
```

```sh
git stash list : 
## 임시저장 파일 목록 확인
```

```sh
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
# 해당 속성의 값을 삭제한다.
```
​

​

## <span style="color:#802548">_12. 태그_</span>
​
```sh
git tag -l : 
# tag를 달은 모든 목록을 보여줌.

git tag [태그] [해쉬값] -m "message" : 
# git tag -l에는 message가 없지만 git show [tag이름]하면 부여한 message가 나옴.

git tag -d [태그이름] 
# 태그 삭제. d는 delete

git push origin --tags :  
# 로컬저장소의 모든 태그를 원격저장소에도 동기화

git push origin --delete [태그이름] : 
# 특정 태그만 원격저장소와 로컬저장소 모두에서 삭제
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
# git의 설정을 연다. 아래와 같이 alias를 선언하고 쭊 써주면 된다. 참고로 저기 pn은 먹히지 않는다. gitconfig는 git에서 인식 가능한 command만을 alias로 설정할 수 있다. pnpm은 또 다른 system이라서 인식되지 않는다.
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



​

https://xtring-dev.tistory.com/entry/Git-%EC%9D%B4%EB%AF%B8-commit%ED%95%9C-%EB%A9%94%EC%84%B8%EC%A7%80-%EC%88%98%EC%A0%95%ED%95%98%EA%B8%B0-%EB%B0%94%EB%A1%9C-%EC%9D%B4%EC%A0%84%EA%B7%B8-%EC%A0%84%EB%A6%AC%EB%AA%A8%ED%8A%B8-commit

 

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

git checkout -- .

## <span style="color:#802548">_18. 이쁘게 커밋 보기_</span>
```sh
git config --global -e(vim ~/.gitconfig)
# .gitconfig 파일이 열린다. 괄호와 동일하다.

[alias]
        hist = log --graph --all --pretty=format:'%C(yellow)[%ad]%C(reset) %C(green)[%h]%C(reset) | %C(white)%s %C(bold red){{%an}}%Creset' --abbrev-commit
# 해당 alias를 추가해준다.

git config --list
# hist라는 alias가 추가되었는지 확인한다.

git hist
```