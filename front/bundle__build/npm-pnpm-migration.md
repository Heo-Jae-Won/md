## <span style="color:#802548">_1. pnpm dependency install_</span>

- 먼저 npm으로 pnpm을 깐다. 
- npm을 쓰지 않으려고 npm으로 pnpm을 까는 게 아이러니지만 어쩔 수 없다. npm이 기본매니저기 때문이다. 
- 그게 싫다면 corepack enable을 사용하라고 공식 사이트에서 이야기 하고 있다. 

```
npm i -g pnpm
```

- 그 다음으로 node_modules를 전부 삭제한다.
- y 누르고 기다렸다가 deleted 뜨면 q누르면 삭제가 완료된다.

```
npx npkill
```

## <span style="color:#802548">_2. npm command 무력화_</span>
- 이제 npm install을 무력화시키는 설정이 필요하다. 
- package.json에서 아래와 같이 preinstall을 추가해주자. 
- 참고로 npm run start같은 것은 된다. npm으로 install하는 게 막힐 뿐이다. 

```js
  "scripts": {
    "preinstall": "npx only-allow pnpm",
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
```

- 그 다음으로  pnpm-workspace.yaml를 만들어 받을 package와 받지 않을 package를 나누는 내용을 추가한다. 
- !가 붙으면 해당 경로의 package는 받지 않는 것이다. 
- 여기서는 어떤 경로든 test라는 폴더가 중간에 들어가면 해당 폴더의 하위는 전부 package를 install하지 않는 설정이다. 

```js
packages:
  # all packages in direct subdirs of packages/
  - 'packages/*'
  # all packages in subdirs of components/
  - 'components/**'
  # exclude packages that are inside test directories
  - '!**/test/**'
```

- 그 뒤에 pnpm import를 통해 lock 파일을 생성한다. 

```
pnpm import
```

- 그냥 pacakge는 버전이 뭐 이상이라면 lock은 버전을 정확히 명시한다.
- pnpm으로 만든 lock yml이 생겼으므로 다른 package manager의 lock파일은 지워준다. 
- 그리고서 이제 다시 pnpm으로 package를 install하면 끝이다.

```
pnpm i
```

## <span style="color:#802548">_3. alias 적용_</span>
- window에서는 아래와 같이 alias를 만든다. 단 이는 powershell 전용이다.
- powershell에서 get-alias로 별칭을 확인해보면 아래와 같이 생겨난 것을 볼 수 있다.
- git bash로 local에서 돌린다면 해당 alias가 먹히지 않는다.
- git bash는 powershell과 또다른 terminal이라 다른 방식의 작업이 요구된다. 

```sh
vim ~/.bashrc
```

- vim이 켜지면 i를 눌러 편집모드로 바꾸고 내용을 적는다. 내용을 다 적었다면 esc를 누르고 :wq를 눌러 저장하고 빠져나간다.
- 그다음으로, 적은 내용을 실시간으로 반영한다. 

```sh
source ~/.bashrc
```

- ~/.gitconfig에 alias를 적는 것은 system command의 별칭이 아니라서 작동하지 않는다. 따라서 bashrc 파일에 적어야 한다. 
- bashrc가 아니라 bash_profile에 적어주는 게 더 좋다. bash_profile이 먼저 있는 게 올바른 순서라고 한다. 


아래 사이트를 참조해서 썼다.

https://supern0va.tistory.com/12

https://pnpm.io/ko/installation
