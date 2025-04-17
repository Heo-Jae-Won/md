## <span style="color:#802548">_CSR에서 SSR 환경으로 전환_</span>

- import 문법은 기본적으로 node_modules에서 가져오는 문법이다.
    - node_modules 폴더가 있다면, 즉 한번만 npm install이 진행되었다면 외부망-내부망을 가리지 않는다.
    - 다만 node.js가 깔려있어야 처음에 install을 할 수 있다만, legacy 폐쇄망 환경에서는 node.js를 깔 수 없었다.
- 따라서 script src로 전환해야하는데, 그를 위해서는 외부에서 script js를 다운받아 반입해야 한다.

```
https://www.jsdelivr.com/package/npm/fabric ---cdn download
https://cdnjs.com/ ---cdn
```

- 원래 CSR 환경에서는 아래와 같이 각 기능들이 전부 분리되어 있는 형태였다.
- call 하는 순서는 신경써야 한다. 부모 함수가 call 된 뒤에야 기능이 활성화되는 자식 함수가 존재한다.
    - 따라서 canvas가 첫번째이며, 그 다음 text를 활성화시키고, 그 다음 font를 활성화시키는 순서다.

```js
//core.js
import initCanvas from "./function/initCanvas";
import initText from "./function/initText.js";
import initFont from "./function/initFont";
.
.
.
const init = () => {
    initCanvas.call(this);
    initText.call(this);
    initFont.call(this);
    .
    .

};

export default init;
```

- 아래는 SSR로 전환한 것이다. CSR과 동일하게 먼저 실행되어야 하는 것부터 부른다.
- 각 기능별로 분리되어 있는 함수들은 전부 한 곳에 집어넣은 형태인데, 이는 고객사의 요구사항이었다.
- OOP처럼 각 책임별로 파일을 분리하지말고 하나의 파일에 모아달라고 요청받았다.
    - OOP원칙을 어기는 일이지만, 요구대로 구현하는 게 우선이었기 때문에 해당 방식으로 구현했다.
    - 각 파일별로 100줄 가량이 한 파일에 합치다보니 3000줄이 넘어서 개발/유지보수에는 바람직하지 않게 되긴했다.

```js
//init.js
const init = function() {

    const initCanvas = function() {
        .
        .
    }

    
  
    const initFont = function() {
        .
        .
    }

    const initText = function () {
        const init = () => {
            initFont({
                fontA: getEl('#fontA'),
            }, this.canvas);

            //eventListener business logic
        }

        init();
    }

    initCanvas.call(this);
    initText.call(this);
    initFont.call(this);
}

export default init;
```

- photoEditor를 이루는 기능을 사용하기 위해서는 기능을 모아둔 init을 불러야한다.
- 이전의 this binding된 함수들은 모두 photoEditor instance를 의미하게 된다. 
    - photoEditor에서 만들어진 field인 canvas 등을 사용할 수 있다는 의미다.
    - 

```js
//photoEditor.js
import init from "init.js";
const PhotoMaker  = () => {

    this.canvas = new fabric.Canvas(canvasId, {
        isDrawingMode: false,
        width: canvasWrap.clientWidth - 261,
        height: canvasWrap.clientHeight,
    });

    init.call(this);

    return this;
}

export default PhotoMaker ;
```

- init.js의 내 initText function의 this.canvas도 PhotoEditor instance의 field다.

```js
const initText = function () {
    const init = () => {
        initFont({
            fontA: getEl('#fontA'),
        }, this.canvas);

        //eventListener business logic
    }

    init();
}
```

- 이제 photoEditor를 new로 생성하여 사용할 수 있다.
- 각 페이지 별 js를 만들게끔 되어있으므로 PhotoEditor 기능이 필요한 페이지는 전부 해당 형태로 new를 호출한다.


```js
//PhotoEditorPage.js
import PhotoMaker from "photoEditor.js";

$(function() {
    new PhotoMaker('canvas');

    //page business logic
    $("#button").click(function() {

    })
});
```

- HTML도 하나로 합쳐달라는 요구가 있어 HTML도 하나로 합쳤다.
- 하나의 PhotoEditor 객체만 존재하므로 편하지만, HTML이 길어져서 수정하기가 어려워지는 단점이 있다.

```html
<!--photoEditor.html-->
<head>
    <script src="PhotoEditorPage.js"></script>
</head>

.
.
.
```




## <span style="color:#802548">_뒤로가기_</span>

- 처음 뒤로가기 로직은 매우 간단했다.

```js
this.undoStack = [];

const saveState = (e) => {

  if(e.isForeGround == true && e.target) {
      this.undoStack.pop();
      const copiedCanvasAsJson = JSON.stringify(this.foregroundCanvas.toObject(["name"]));
      this.undoStack.push(copiedCanvasAsJson);
  }

}

this.foregroundCanvas.on("object:modified", saveState);
```


- 하지만 텍스트 관련 기능이 계속 추가되면서 복잡해지기 시작했다.
- 특히 차후에 추가된 textPlaceHolder 요구사항이 문제였다.
    - text에 값이 없어지면 textPlaceholder를 보여달라고 했는데, canvas api에는 html과 같은 자동 placeholder가 없었다. 
    - 따라서 수동으로 전부 event를 조작해야했는데, 그게 뒤로가기와 엮어 문제를 일으켰다.
    - text 관련 instance가 6개씩 추가되면서 문제가 되었다.
    - text 값의 변화 혹은 폰트의 변화 자체는 1개씩 인식되어 되돌린다.
        - 하지만 textPlaceHolder의 경우 6개의 instance가 한꺼번에 추가되는데, 이 경우 1개만 되돌리면 안되었다.
        - 실제 유저에게 있어 textHolder는 뒤로가기로서 취급되면 안되기 때문이다.
        - 즉 그 경우를 구분하여 어쩔 때는 6개를, 어쩔 떄는 1개를 되돌려야 했다.
- 이를 반영하기 위해서 textFlag 변수를 도입했다.

```js
const initText = function () {
    const init = () => {
        initFont({
            fontA: getEl('#fontA'),
            fontB: getEl('#fontB'),
        }, this.canvas);

        const leftTextPlaceHolder = new fabric.Text(leftDefaultText, {textFlag: "initial"})
        const rightTextPlaceHolder = new fabric.Text(rightDefaultText, {textFlag: "initial"})
        const centerTextPlaceHolder = new fabric.Text(centerDefaultText, {textFlag: "initial"})
        .
        .

    }

    init();
}
```


- 하지만 placeholder도 맨 처음 6개가 한꺼번에 생성되는 경우와, 입력했다가 다시 입력값을 지워야하는 경우를 구분해야 했다.
- 따라서 각 경우를 나누어 별도의 flag 변수를 사용하는 것으로 진행했다.
- 뒤로가기 시에는 해당 flag 변수를 확인하며 진행한다.

```js
// textPlaceHolder regenerate eventListener
this.canvas.on('text:editing:entered', (e) => {
    .
    .
    .

    if (e.target.name.include('text_left')) {

        if (!rightText.text && !rightPlaceholder) {
            this.canvas.add({...rightPlaceholder, textFlag: "center"});
        }
        .
        .
        .
    }

})
```




## <span style="color:#802548">_회전에 따른 카드 여백체크 불가능 문제_</span>

- 카드이미지에 여백이 없게 이미지를 전부 덧씌웠는지 확인해달라는 요구사항이 있었다.
- 해당 사항 구현을 간단하게 생각했으나, 그리 간단하지 않았다.
- 회전을 하게 되면, left, top 값이 당연히 회전된 객체에 맞춰 정해지는 거라고 생각했다.
- 그런데, 전혀 그렇지 않았다. 원래의 left, top값으로 찍힌 좌표가 원하는대로 이동하지 않았다.
- 그런 이유로 회전된 이미지의 경우, 여백을 체크하는 로직도 그러다보니 전부 고장났다.
- left, top 값을 일일이 전부 각도에 맞춰 바꿔주는 것을 시도했다.

```js
const imageObj = this.canvas.getObjects().find((obj) => obj.name?.includes('image'))

let imgLeft = imageObj.left
let imgTop = imageObj.top
if(imageObj.angle === 0 || imageObj.angle === 360 || imageObj.angle === -360 ) {
    imageObj.left = imageObj.oCoords.tl.x;
    imageObj.top = imageObj.oCoords.tl.y;
} else if(imageObj.angle === -90) {
    imageObj.left = imageObj.oCoords.tr.x;
    imageObj.top = imageObj.oCoords.tr.y;
} else if(imageObj.angle === -180) {
    imageObj.left = imageObj.oCoords.br.x;
    imageObj.top = imageObj.oCoords.br.y;
} else if(imageObj.angle === -270) {
    imageObj.left = imageObj.oCoords.bl.x;
    imageObj.top = imageObj.oCoords.bl.y;
}
```                  


- 또한 rotate 시 매번 -90도를 하는 것 때문에, -90 + 360n으로 무한하게 angle 값이 뻗어나가고 있었다. 
- 해당 경우에는 if문을 구성하는 게 불가능하기 때문에, -360도와 +360도 사이에서 값이 변동하게 만들어야 했다.
- 그래서 아래와 같이 angle이 한바퀴돌아 원래상태로 돌아오게 되면 0으로 보정해주는 작업을 진행했다.
- 다행히 0인 상태에서 회전을 진행해도 아무 문제가 없었다.

```js
if (activeSelection) {
      // 현재 중심점 구하기
      const center = activeSelection.getCenterPoint();
      if (activeSelection.angle === -360) {
        activeSelection.angle += 360;
      } else if(activeSelection.angle === +360) {
        activeSelection.angle -= 360;
      }

      // 회전 설정
      activeSelection.set({
        originX: "center",
        originY: "center",
        angle: activeSelection.angle - 90
      });
}
```

- 하지만 위와 같은 코드는 js로 90도씩 돌렸을 때만 유효하다.
- 근본적으로 사용자가 마우스로 돌리는 event에는 소용이 없다.
- 그래서 고민해봤지만, 도저히 답이 나오지 않아 구글링을 하였다.
- 구글링으로도 도저히 못찾겠어서, 결국 CHATGPT에 물어봐서 해결했다.
    - 해당 의도만 기술한다고 되지는 않았다. 해당 의도를 만족하는 알고리즘이 있는 지 물어야 알고리즘을 알려준다.
    - 알고리즘을 알았다면 그 알고리즘의 구현 방법에 대해 질문한다.
    - 해당 알고리즘은 point-in-polygon algorithm이다.

```js
const petImage = this.canvas.getObjects().find((obj) => obj.name?.includes('image'))
const cardImage = this.canvas.getObjects().find((obj) => obj.name?.includes('background'))
const doesCardCoveredFullyByPetImage = isCardInsidePet(cardImage.getCoords(),  petImage.getCoords())

if(!doesCardCoveredFullyByPetImage)  {
    alert('空白はNGです')
    return; 
}

function isCardInsidePet(innerCoords, outerCoords) {
    return innerCoords.every(({ x, y }) => {
        let inside = false;
        for (let i = 0, j = outerCoords.length - 1; i < outerCoords.length; j = i++) {
            let xi = outerCoords[i].x, yi = outerCoords[i].y;
            let xj = outerCoords[j].x, yj = outerCoords[j].y;

            let intersect = ((yi > y) !== (yj > y)) &&
                (x < (xj - xi) * (y - yi) / (yj - yi) + xi);

            if (intersect) inside = !inside;
        }
        return inside;
    });
}
```

- every이기 때문에 모든 점이 polygon(펫 이미지) 안에 존재해야, 즉 inside 변수가 true여야 true를 return한다.
- for문을 통해 펫 이미지의 모든 점에 대해서 순회를 시작한다. for문에 변수를 두 개를 사용할 수 있다.
- intersect를 통해 카드 이미지와의 교차 여부를 검증한다. 
    - 카드 이미지에서 쏴서 펫 이미지와 짝수 번 교차하면 카드에 여백이 있다.
    - 홀수 번 교차하면 카드에 여백이 없다.

```js
function isCardInsidePet(innerCoords, outerCoords) {
    return innerCoords.every(({ x, y }) => {
        let inside = false;
        for (let i = 0, j = outerCoords.length - 1; i < outerCoords.length; j = i++) {
            let xi = outerCoords[i].x, yi = outerCoords[i].y;
            let xj = outerCoords[j].x, yj = outerCoords[j].y;

            let intersect = ((yi > y) !== (yj > y)) &&
                (x < (xj - xi) * (y - yi) / (yj - yi) + xi);

            if (intersect) inside = !inside;
        }
        return inside;
    });
}
```

