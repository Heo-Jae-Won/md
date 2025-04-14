## <span style="color:#802548">_import에서 script 파일로의 전환_</span>

- import 문법은 기본적으로 node_modules에서 가져오는 문법이다.
- node_modules 폴더가 있다면, 즉 한번만 npm install이 진행되었다면 외부망-내부망을 가리지 않는다.
- 다만 node.js가 깔려있어야 처음에 install을 할 수 있다.
- legacy 환경에서는 당연히 node가 없고, npm을 쓸 수 없는 환경이므로 script로 전환해야 한다.
- script로 전환하려면 cdn으로 js를 다운받아야 한다.
- 아래와 같이 script를 받는다.

```
https://www.jsdelivr.com/package/npm/fabric ---cdn
https://cdnjs.com/ ---cdn
```

- 아래는 npm을 이용한 ES6 js다.
- import를 받을때는 자신이 원하는 변수로 받아올 수 있다.
- 아래는 lodash를 _로 받아오지만, aa로 받아오는 것도 가능하다.

```html
<!DOCTYPE html>
<html lang="kr">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>포토에디터</title>
    <link rel="stylesheet" href="src/assets/style/css/swiper-bundle.min.css">
    <link rel="stylesheet" href="src/assets/style/css/style.css"/>
    <link rel="stylesheet" href="src/assets/style/scss/style.scss"/>
  </head>

  <body id="app">
    <script type="module" src="/src/index.js"></script>
  </body>
</html>
 
<!--js-->
<script>
import _ from "lodash";

const initIcon = function () {
  .
  .
  .
  img.name = _.uniqueId("icon_");

};

export default InitLayer;
</script>
```


- 아래는 위의 ES6 용도를 legacy 환경으로 전환한 것이다.
- 가장 최상위 html에 script import를 해준다.
- script type module과 src로 구분해서 받아와야 한다.
- 둘의 구분점은 아래와 같다.
  - src로 import한 것은 순수 js이며, 전역 변수로 존재한다.
    - library js 내부에서 import, export를 쓸수가 없다.
  - module로 import한 것은 module이며, import와 export를 지원한다.
    - library js 안에서 import를 쓸 수 있다.
    - library js 안에서 export한 것이 있다면, 다른 js에서도 import로 쓸 수 있다.
- script로 가져온 경우, import처럼 자기 맘대로 변수명을 지정할 수가 없다.
  - api documentation을 구글에 쳐서 들어가 해당 namespcae를 가져와야 한다.
  - 대충 용례를 보면 변수.method()로 하는데, 변수명이 namespace라고 보면 된다.
  - Jqueryt는 $, lodash는 _, tensorflow는 tf, tensorflow/coco-ssd는 cocoSsd, exifr은 exifr등...

```html
<!DOCTYPE html>
<html lang="kr">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>포토에디터</title>
    <link rel="stylesheet" href="src/assets/style/css/swiper-bundle.min.css">
    <link rel="stylesheet" href="src/assets/style/css/style.css"/>
    <link rel="stylesheet" href="src/assets/style/scss/style.scss"/>
    <script type="module" src="src/assets/js/tf.js"></script> 
    <script type="module" src="src/assets/js/coco-ssd.js"></script>
    <script src="src/assets/js/fabric.js"></script>
    <script src="src/assets/js/lodash.js"></script>
    <script type="module" src="src/assets/js/exifr-min.js"></script>
    
  </head>

  <body id="app">
    <script type="module" src="/src/index.js"></script>
  </body>
</html>

<!--js-->
<script>
const classifyImage = async (imgObj) => {
    // cocossd 모델 로드
    if (!loadedModel) {
        // loadedModel = await cocoSsd.load();
        loadedModel = await cocoSsd.load({
         .
         . 
        })
    }
}
</script>
```


- script import를 가져올 때, export나 import가 문법을 지원하지 않는 library는 src로 가져온다.

```html
<script src="/conts/photo/libjs/fabric.js"></script>
<script src="/conts/photo/libjs/tf.js.js"></script>
<script src="/conts/photo/libjs/coco-ssd.js"></script>
<script src="/conts/photo/libjs/lodash.js"></script>
```

- libray js 내에서 node 환경인지, AMD 환경인지, browser환경(대부분의 내부망-외부인터넷 연결X, 노드X, bundler x- 쓰는 legacy들)인지 검사하기 때문이다.
  - node를 쓰면 require("@tensorflow/tfjs-converter")로 해당 library를 가져온다. 마치 maven처럼 연관된 모든 library를 가져온다고 보면 된다.
  - require.js도 이와 마찬가지다.
  - browser 환경에서는 window 전역 변수에 넣어서 사용한다. self가 window다.
    - 내부망이어도 browser 환경이다. 외부망이 안된다고 browser가 아닌 것은 아니다.
    - var self: Window & typeof globalThis라는 것을 볼 수 있다.
    - https://developer.mozilla.org/ko/docs/Web/API/Window/self 를 참조하자.

```js
!(function (e, a) {
  // Check if we're in a CommonJS (Node.js) environment
  "object" == typeof exports && "undefined" != typeof module
    ? a(
        exports,
        require("@tensorflow/tfjs-converter"),
        require("@tensorflow/tfjs-core")
      )
    // Check if we're in an AMD environment(require.js)
    : "function" == typeof define && define.amd
    ? define(
        ["exports", "@tensorflow/tfjs-converter", "@tensorflow/tfjs-core"],
        a
      )
    // Otherwise, assume we're in a browser environment and use browser globals
    : a(((e = e || self).cocoSsd = e.cocoSsd || {}), e.tf, e.tf);
}(this, function (exports, tfjsConverter, tfjsCore) {
  // Module implementation goes here
}));

```

- script import를 가져올 때, export나 import가 있는 library는 type = module로 가져온다.

```html
<script type="module" src="/conts/photo/publishingjs/swiper.js"></script>
```

- 핵심 module js를 일단 import하면 다른 모든 것들을 import할 필요는 없다.
- core.js를 import했기 때문에, 그 안에서 쓰이는 import는 전부 html에 넣을 필요가 없다.
- 따라서 initRetouch.js 등을 src="initRetouch.js"등으로 html에서 가져올 필요가 없다.
- legacy환경이기 때문에, vite 설정이 없다.
  - 따라서 뒤에 .js 생략이라던가,
  - @ 경로 인식이라던가,
  - 폴더 뒤에 아무것도 없으면 index.js를 가져온다던가 하는 부분이 하나도 없다.
  - 다시말해 전부 다 써줘야 한다는 뜻이다. 
    - initFilter가 아니라 initFilter.js
    - @가 아니라 /conts/photo/~
    - util/가 아니라 util/index.js

```html
<!--allone.html-->
<head>
  <script type="module" src="core.js"></script>
  <!-- <script type="module" src="initRetouch.js"></script> -->
</head>

.
.
.


<!--core.js-->
import initRetouch from "./function/initRetouch.js";
import initResize from "./function/initResize.js";
import initFilter from "./function/initFilter";
// import initBrush from "./function/initBrush";
import initFlip from "./function/initFlip";
import initErase from "./function/initErase";
// import initMove from "./function/initMove";
import initReset from "./function/initReset";
import { accordion } from "./assets/js/accordion";
import { swiper } from "./assets/js/swiper";
import { stickerSwiper } from "./assets/js/stickerSwiper";
import { modal } from "./assets/js/modal";
import { TOOL } from "./utils/consts";
import backgroundRemove from "/src/function/backgroundRemove";
import initColor from "./function/initColor";
import { activeBtn, getEl } from "./utils/index"
import initCanvas from "./function/initCanvas";
import initRotate from "./function/initRotate";
const init = () => {
        this._initCanvas = initCanvas.call(this);
        this._initImageUpload = InitImageUpload.call(this);
        this._initActiveSelection = InitActiveSelection.call(this);
        this._intiControllViewport = ControllViewport.call(this);
        this._initPreview = Preview.call(this);
        this._initUndo = initUndo.call(this);
        this._initLayer = InitLayer.call(this);
        this._initCrop = InitCrop.call(this);
        this._initText = initText.call(this);
        this._initIcon = initIcon.call(this);
        this._initRetouch = initRetouch.call(this);
        this._initResize = initResize.call(this);
        this._initFilter = initFilter.call(this);
        // this._initBrush = initBrush.call(this);
        this._initFlip = initFlip.call(this);
        // this._initMove = initMove.call(this);
        this._initReset = initReset.call(this);
        this._initConfirmation = Confirmation.call(this);
        this._initErase = initErase.call(this);
        this._initRotate = initRotate.call(this);

        // 에디터 일 경우만 배경 제거, 배경 색상 변경 호출
        if(window.location.pathname == '/diyEditor') {
            this._backgroundRemove = backgroundRemove.call(this);
            this._initColor = initColor.call(this);
        }
    };
```

- 사실 저 불러오는 여러가지 기능 별 함수도 전부 합쳐줄 수 있다.
- 얼탱이 없을 수 있지만, 신한카드 같은 병신같은 곳은 유지보수를 생각하지 않고 일단 함수를 뭉쳐달라고 한다.
- 각 함수에서 썼던 모든 import를 전부 한 곳에 몰아넣고, 함수도 몰아 넣는다.
- 그리고 그 함수집합을 감싸는 함수 하나를 만든다.
  - 여기서는 one이라고 이름지었다.

```js
import ~~ from "~~";

const one = function() {
  let loadedModel = null;

  //backgroundRemove의 원래 내용...
  //export default를 지우고 가져온다.
  //import는 전부 위에다가 넣어준다.
  const backgroundRemove = function() {
    .
    .
  }
  
  const initCanvas = function() {
    .
    .
  }
  .
  .
  .


}

export default one;
```


- 함수를 선언하는 것만으로 끝이 아니다. 불러줘야 한다.
- 불러줄 떄는 함수가 먼저 실행되어야 하는 것부터 부른다.
- 그 떄 .call(this)를 붙여서 불러줘야 한다. 아니면 this binding이 안 된다.

```js
import ~~ from "~~";

const one = function() {
  let loadedModel = null;

  //backgroundRemove의 원래 내용...
  //export default를 지우고 가져온다.
  //import는 전부 위에다가 넣어준다.
  const backgroundRemove = function() {
    .
    .
  }
  
  const initCanvas = function() {
    .
    .
  }
  .
  .
  .

  initCanvas.call(this);
  ImageUpload.call(this);
  initActiveSelection.call(this);
  ControllViewport.call(this);
  .
  .
}

export default one;
```

- 그럼 core.js 호출 시에 one을 불러오면 된다.

```js
import one from "one.js";
const PhotoEditor = () => {
  const init = () => {
  .
  .

  one.call(this);
}

  init();

  return this;
}

export default PhotoEditor;
```

- 어차피 devon은 html render 시에 함수를 발동시킨다.
- 따라서 render 시에 발동시킬 수 있게 아래같이 써준다.


```js
//ㅈ같은 이름의 mobfm038.js
import PhotoEditor from "one.js";
import accordion from "accordion.js";
import swiper from "swiper.js";
import activeBtn from "activeBtn.js";

$(function() {
  new PhotoEditor('canvas');
  accordion();
  swiper();
  .
  .
  activeBtn();
});

/*devon 식은 아래처럼 했었다.
(function() {
  var $svc = $.get('svc');

  $svc.view('mobfm038',function() {
    var vo = $svc.bind({name:'mobfm038'});
    vo.render(function(){
      one.call(this);
      accordion();
      swiper();
      .
      .
      activeBtn();
    })
  })

})
*/
```



## <span style="color:#802548">_뒤로가기_</span>

- 뒤로가기는 이전에 구현되어있었으나, 폰트색깔을 바꾸고 이미지가 같이 바뀌게 되면서 뒤로가기가 좀 더 복잡해졌다
- 폰트가 바뀌면서 이미지가 바뀌지만 뒤로가기 스택에는 이전의 이미지가 저장되지 않았기 때문이다.
- 따라서 아래와 같이 이미지용 스택을 따로 만들어준다.
  - 따로 만드는 게 편한 이유는 따로 관리하지 않으면 기존의 스택과 겹쳐 어느 시점으로 뒤돌아가야 할 지 판단하기 어렵기 때문이다.
  - 별 건 아니고 이전의 stack을 되돌리는 코드와 동일하게 foreground stack을 되돌리는 코드를 추가한 것이다.
  - 이전엔 폰트만 저장되던 상황에서 이제는 이미지와 폰트가 같이 저장되고, 같이 되돌려진다.
  
```js
this.undoForegroundStack = []; // foreground 캔버스 undo스택
let redoForegroundStack = [];
 
 // 폰트 색상 변경 시 추가되는 undoForegroundStack pop
const saveState = (e) => {

  if(e.isForeGround == true && e.target) {
      this.undoForegroundStack.pop();
      const jsonForeground = JSON.stringify(this.foregroundCanvas.toObject(["name"]));
      this.undoForegroundStack.push(jsonForeground);
  }


  if (
      (e.target &&
      e.target.name &&
      e.target.name != 'layerIgnore') && // 브러쉬에서 객체가 10개 넘어서 자동으로 지울 때 객체 name = 'layerIgnore'
      !isUndo &&
      !isRedo &&
      !this.isReset &&
      !this.isCrop
  ) {
      // 뒤로가기 횟수 제한
      if (this.undoStack.length > maxLen) {
          let newStack = [];
          for (let idx in this.undoStack) {
              if (idx > 0) newStack.push(this.undoStack[idx]);
          }
          this.undoStack = newStack;
      }

      // foreground 뒤로가기 횟수 제한.
      if (this.undoForegroundStack.length > maxLen) {
          let newStack = [];
          for (let idx in this.undoForegroundStack) {
              if (idx > 0) newStack.push(this.undoForegroundStack[idx]);
          }
          this.undoForegroundStack = newStack;
      }

      const json = JSON.stringify(this.canvas.toObject(["name"]));
      this.undoStack.push(json);

      const jsonForeground = JSON.stringify(this.foregroundCanvas.toObject(["name"]));
      this.undoForegroundStack.push(jsonForeground);
  }

}


this.foregroundCanvas.on("object:modified", saveState); //foreg
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

- 그런데 문제가 있었다. 
- 회전이후 scaledWidth와 Height가 변하지 않는다는 사실이었다.
- 그래서 각도에 따라 Right, bottom값을 분기처리했다.
  - -90도, 즉 왼쪽으로 90도면 Right는 getScaledWidth를 가져와 imgLeft를 더한다.
  - 만약 -180도, 즉 왼쪽으로 90도면 Right는 getScaledHeight를 가져와 imgTop를 더한다.

```js
const imageObj = this.canvas.getObjects().find((obj) => obj.name?.includes('image'));

let imgLeft = imageObj.left
let imgTop = imageObj.top
let imgRight = null;
let imgBottom = null;

if (imageObj.angle === 0 || imageObj.angle === 360 || imageObj.angle === -360 ) {
  imgLeft = imageObj.oCoords.tl.x;
  imgTop = imageObj.oCoords.tl.y;
  imgRight = imageObj.getScaledWidth() + imgLeft;
  imgBottom = imageObj.getScaledHeight() + imgTop;
} else if(imageObj.angle === -90) {
  imgLeft = imageObj.oCoords.tr.x;
  imgTop = imageObj.oCoords.tr.y;
  imgRight = imageObj.getScaledHeight() + imgLeft;
  imgBottom = imageObj.getScaledWidth() + imgTop;
} else if(imageObj.angle === -180) {
  imgLeft = imageObj.oCoords.br.x;
  imgTop = imageObj.oCoords.br.y;
  imgRight = imageObj.getScaledWidth() + imgLeft;
  imgBottom = imageObj.getScaledHeight() + imgTop;
} else if(imageObj.angle === -270) {
  imgLeft = imageObj.oCoords.bl.x;
  imgTop = imageObj.oCoords.bl.y;
  imgRight = imageObj.getScaledHeight() + imgLeft;
  imgBottom = imageObj.getScaledWidth() + imgTop;
} else if(imageObj.angle === 90) {
  imgLeft = imageObj.oCoords.bl.x;
  imgTop = imageObj.oCoords.bl.y;
  imgRight = imageObj.getScaledWidth() + imgLeft;
  imgBottom = imageObj.getScaledHeight() + imgTop;
} else if(imageObj.angle === 180) {
  imgLeft = imageObj.oCoords.br.x;
  imgTop = imageObj.oCoords.br.y;
  imgRight = imageObj.getScaledHeight() + imgLeft;
  imgBottom = imageObj.getScaledWidth() + imgTop;
} else if(imageObj.angle === 270) {
  imgLeft = imageObj.oCoords.tr.x;
  imgTop = imageObj.oCoords.tr.y;
  imgRight = imageObj.getScaledWidth() + imgLeft;
  imgBottom = imageObj.getScaledWidth() + imgTop;
} 
```


- 하지만 위와 같은 코드는 js로 90도씩 돌렸을 때만 유효하다.
- 근본적으로 사용자가 마우스로 돌리는 event에는 소용이 없다.
- 그래서 고민해봤지만, 도저히 답이 나오지 않아 구글링을 하였다.

```js
const imageObj = this.canvas.getObjects().find((obj) => obj.name?.includes('image'))
const cardObj = this.canvas.getObjects().find((obj) => obj.name?.includes('background'))
const isAllInside = isPointInsidePolygon(imageObj.getCoords(),  cardObj.getCoords())

if(!isAllInside)  {
    alert('카드 영역 내 여백이 생겨서는 안됩니다')
    return; 
}

function isPointInsidePolygon(imgCoords, cardCoords) {
    
    let insideCnt = 0;

    cardCoords.forEach(point => {
        let x = point.x
        let y = point.y;
        let inside = false;

        for (let i = 0, j = imgCoords.length - 1; i < imgCoords.length; j = i++) {
            let xi = imgCoords[i].x, yi = imgCoords[i].y;
            let xj = imgCoords[j].x, yj = imgCoords[j].y;

            let intersect = ((yi > y) != (yj > y)) &&
                            (x < (xj - xi) * (y - yi) / (yj - yi) + xi);
        
            if(intersect) {
                inside = !inside;
            }
        }

        if(inside) {
            insideCnt++;
        }
    })

    if(insideCnt === 4) {
        return true
    }

    return false;
  }
```

- 이제 cropZone 문제도 풀렸다.
- cropZone이 이미지 내에서만 이동하게 하는 코드를 그대로 넣고, left, top만 angle에 맞게 변경시켜준다.


## <span style="color:#802548">_image 내에서만 cropzone 움직이게 하기_</span>
- image와 동일한 크기의 rect를 만들어서 움직이는데, 그게 image zone에서만 움직이게 해달라는 이야기가 있었다.
- 그걸 구현하지 못해서 도움을 구했다.
- 핵심 기획은 회전이 된 경우는 


```js
this.canvas.on('object:moving', (e) => {
    const obj = e.target;

    if (!obj || obj.name !== 'cropZone') {
        return;
    }

    const img = getCurImage(this.canvas);

    const imgLeft = img.left;
    const imgTop = img.top;
    const imgWidth = img.getScaledWidth();
    const imgHeight = img.getScaledHeight();

    //cropzone rect left, top 값을 이미지의 각도에 맞춰 수정
    const {x:rx, y:ry} = rotatePoint(obj.left, obj.top, imgLeft, imgTop, -img.angle);
    obj.left = rx;
    obj.top = ry;

    if (obj.left < imgLeft) {
        obj.left = imgLeft;
    }
    if(obj.top < imgTop) {
        obj.top = imgTop;
    }

    if (obj.left + obj.getScaledWidth() > imgLeft + imgWidth) {
        obj.left = imgLeft + imgWidth - obj.getScaledWidth();
    }
    if (obj.top + obj.getScaledHeight() > imgTop + imgHeight) {
        obj.top = imgTop + imgHeight - obj.getScaledHeight();
    }

    const {x, y} = rotatePoint(obj.left, obj.top, imgLeft, imgTop, img.angle);
    obj.left = x;
    obj.top = y;

    obj.setCoords();
})

function rotatePoint(px, py, ox, oy, angle) {    
    const radians = angle * (Math.PI / 180);
    const translatedX = px - ox;
    const translatedY = py - oy;

    const rotatedX = translatedX * Math.cos(radians) - translatedY * Math.sin(radians);
    const rotatedY = translatedX * Math.sin(radians) + translatedY * Math.cos(radians);

    const finalX = rotatedX + ox;
    const finalY = rotatedY + oy;

    return { x: finalX, y: finalY}
}
```

- 이미지가 scaling된 상태일 수 있기 때문에, width와 height는 scaled된 값을 구해야 한다.

```js
const img = getCurImage(this.canvas);

const imgLeft = img.left;
const imgTop = img.top;
const imgWidth = img.getScaledWidth();
const imgHeight = img.getScaledHeight();
```

- cropzone rect left, top 값을 각도에 맞춰 수정해준다.
- 여기서 -angle로 전환하는 이유는 image를 (0,0)으로 맞춰 계산할 때 돌린 각도만큼 반대로 돌려야 하기 때문이라고 한다.

```js
//cropzone rect left, top 값을 이미지의 각도에 맞춰 수정
const {x:rx, y:ry} = rotatePoint(obj.left, obj.top,imgLeft, imgTop, -img.angle);
obj.left = rx;
obj.top = ry;
```

- Math의 cos, sin 함수는 반드시 degree가 아닌 radians 형태의 각도로 넣어야 한다.
- 따라서 degree를 radians 값으로 변환해야 한다.
    - 360도가 2 * PI * radians이므로, 1도는 PI * radians /180이다.
    - angle만큼 곱해주면 angle만큼의 radians 값을 얻을 수 있다.
- ox, oy는 image 객체의 left, top 값이다.
- px, py는 rect 객체의 left, top 값이다.
- rect 객체의 left, top 값에서 image 객체의 값을 빼는 이유는 image 객체의 left, top을 (0,0)이라는 점으로 놓기 위해서다.
    -  그렇게 하면 rotation 계산이 매우 간결해지기 때문이다. 따라서 image (left, top)이 (0,0) 기준으로 변환된 rect X, Y 좌표값을 얻어낸다.
    -  거기서부터 회전된 radians 값을 계산해서 다시한번 원점에서의 차이만큼의 좌표를 계산해낸다. 식은 그냥 외우는 게 나을듯 하다.
    - 그리고 이를 원래 image의 (left, top)에 붙이면 degree만큼 돌아간 rect의 좌표값을 얻어낼 수 있다.

```js
function rotatePoint(px, py, ox, oy, angle) {    
    const radiansConvertedFromDegree = angle * (Math.PI / 180);
    const translatedX = px - ox;
    const translatedY = py - oy;

    const rotatedX = translatedX * Math.cos(radiansConvertedFromDegree) - translatedY * Math.sin(radiansConvertedFromDegree);
    const rotatedY = translatedX * Math.sin(radiansConvertedFromDegree) + translatedY * Math.cos(radiansConvertedFromDegree);

    const finalX = rotatedX + ox;
    const finalY = rotatedY + oy;

    return { x: finalX, y: finalY}
}
```

- boundary 안에서만 움직이게끔 하는 코드다.
- 이미지가 마치 (0,0)인 상황에서 rotate되지 않은 상황으로 만들어서 경계를 넘겼는지 계산하는 것이다.

```js
 if (obj.left < imgLeft) {
    obj.left = imgLeft;
}
if (obj.top < imgTop) {
    obj.top = imgTop;
}
if (obj.left + obj.getScaledWidth() > imgLeft + imgWidth) {
    obj.left = imgLeft + imgWidth - obj.getScaledWidth();
}
if (obj.top + obj.getScaledHeight() > imgTop + imgHeight) {
    obj.top = imgTop + imgHeight - obj.getScaledHeight();
}
```

- angle을 -로 돌려서 계산했던 행위를 되돌린다.
- rect의 left, top 값을 원래대로 되돌리고 실제 좌표계에 반영해준다.

```js
const {x, y} = rotatePoint(obj.left, obj.top, imgLeft, imgTop, img.angle);
obj.left = x;
obj.top = y;

obj.setCoords();
```

- scaling의 경우에는 코드가 약간 달라진다.
- 다만 아직 오류가 있다. 위로 당기면 아래 rect가 커진다.

```js
this.canvas.on('object:scaling', (e) => {
    const obj = e.target;

    if (!obj || obj.name !== 'cropZone') {
        return;
    }

    const img = getCurImage(this.canvas);

    const imgLeft = img.left;
    const imgTop = img.top;
    const imgWidth = img.getScaledWidth();
    const imgHeight = img.getScaledHeight();

    // Calculate the object's scaled dimensions and position
    const scaledWidth = obj.getScaledWidth();
    const scaledHeight = obj.getScaledHeight();

    // Get object's current position
    const { x: objLeft, y: objTop } = obj.getPointByOrigin('left', 'top');

    // Rotate point to the image's coordinate system
    let { x: rx, y: ry } = rotatePoint(objLeft, objTop, imgLeft, imgTop, -img.angle);

    // Boundary conditions
    if (rx < imgLeft) {
        rx = imgLeft;
    }
    if (ry < imgTop) {
        ry = imgTop;
    }
    if (rx + scaledWidth > imgLeft + imgWidth) {
        obj.scaleX = (imgLeft + imgWidth - rx) / obj.width;
    }
    if (ry + scaledHeight > imgTop + imgHeight) {
        obj.scaleY = (imgTop + imgHeight - ry) / obj.height;
    }

    // Rotate point back to the canvas coordinate system
    let { x, y } = rotatePoint(rx, ry, imgLeft, imgTop, img.angle);
    obj.setPositionByOrigin(new fabric.Point(x, y), 'left', 'top');

    // Force the crop zone to update its coordinates and scaling
    obj.setCoords();
});
```





## <span style="color:#802548">_미리보기 배율을 늘리면 여백 검사가 작동하지 않는 문제 해결_</span>

- 미리보기 배율을 늘리면 여백 검사가 작동하지 않는 문제가 있었다.
- 원인은 viewport를 설정하는 부분에 있었다.

```js
 const setViewport = () => {
  const zoomValue = values[viewport.value] * 0.01;
  txtElem.innerText = `${values[viewport.value]} %`;

  // 현재 뷰포트 중심 위치 저장
  const viewportCenter = {
    x: this.canvas.width / 2,
    y: this.canvas.height / 2,
  };

  // 현재 뷰포트 중심 좌표를 캔버스 좌표계로 변환
  const canvasCenter = new fabric.Point(viewportCenter.x, viewportCenter.y);
  const canvasCenterInverted = fabric.util.transformPoint(
    canvasCenter,
    fabric.util.invertTransform(this.canvas.viewportTransform)
  );

  // 뷰포트를 중심으로 확대/축소
  this.canvas.setZoom(zoomValue);
  this.foregroundCanvas.setZoom(zoomValue);

  // 현재 뷰포트 중심 좌표를 업데이트된 캔버스 좌표계로 변환
  const updatedCanvasCenter = fabric.util.transformPoint(
    canvasCenterInverted,
    this.canvas.viewportTransform
  );

  // 뷰포트 변환 행렬 업데이트
  this.canvas.viewportTransform[4] +=
    viewportCenter.x - updatedCanvasCenter.x;
  this.canvas.viewportTransform[5] +=
    viewportCenter.y - updatedCanvasCenter.y;

  this.foregroundCanvas.viewportTransform[4] +=
    viewportCenter.x - updatedCanvasCenter.x;
  this.foregroundCanvas.viewportTransform[5] +=
    viewportCenter.y - updatedCanvasCenter.y;

  this.canvas.requestRenderAll();
  this.foregroundCanvas.requestRenderAll();
};
```

- 아래에서 이미지를 중앙에 노출시키기 위해 x, y축 값을 변환하는데서 문제가 생겼었다.
- 이게 전부 수동으로 이뤄지는 작업이라 좌표값에 문제가 생겼던 셈이다.
- 배율을 늘리면 이미지가 밖으로 튕겨져 나가고 가운데로 들어오지 않아 추가됐던 코드로 보인다.

```js
const canvasCenter = new fabric.Point(viewportCenter.x, viewportCenter.y);
const canvasCenterInverted = fabric.util.transformPoint(
  canvasCenter,
  fabric.util.invertTransform(this.canvas.viewportTransform)
);

// 뷰포트를 중심으로 확대/축소
this.canvas.setZoom(zoomValue);
this.foregroundCanvas.setZoom(zoomValue);

// 현재 뷰포트 중심 좌표를 업데이트된 캔버스 좌표계로 변환
const updatedCanvasCenter = fabric.util.transformPoint(
  canvasCenterInverted,
  this.canvas.viewportTransform
);

// 뷰포트 변환 행렬 업데이트
this.canvas.viewportTransform[4] +=
  viewportCenter.x - updatedCanvasCenter.x;
this.canvas.viewportTransform[5] +=
  viewportCenter.y - updatedCanvasCenter.y;

this.foregroundCanvas.viewportTransform[4] +=
  viewportCenter.x - updatedCanvasCenter.x;
this.foregroundCanvas.viewportTransform[5] +=
  viewportCenter.y - updatedCanvasCenter.y;

this.canvas.requestRenderAll();
this.foregroundCanvas.requestRenderAll();
```

- 똑같이 이미지가 가운데로 오면서, 좌표 값이 정확하게 계산되게끔 하는 방식으로 바꿨다.
- 역시 fabric.js에서 제공하는 api를 활용해야했다.
- 캔버스의 줌포인트를 늘 중앙에 놓으면 이미지가 몇 배율이든 관계없이 중앙에 온다.
- 거기다가 좌표 계산도 정확하게 이뤄진다.

```js
const setViewport = () => {
  const zoomValue = values[viewport.value] * 0.01;
  txtElem.innerText = `${values[viewport.value]} %`;

// 현재 뷰포트 중심 위치 저장
  const viewportCenter = {
    x: this.canvas.width / 2,
    y: this.canvas.height / 2,
  }; 

  this.canvas.zoomToPoint(viewportCenter, zoomValue);
  this.foregourndCanvas.zoomToPoint(viewportCenter, zoomValue);
  this.canvas.renderAll();
  this.foregroundCanvas.renderAll();
}
```


## <span style="color:#802548">_캔버스의 크기에 관계없이 background 카드에 맞게 text 띄우기_</span>

- 캔버스는 반응형이라 화면에 따라 캔버스의 크기가 달라진다. 
- 따라서 직접 하드코딩으로 주는 게 아니라, 관계없이 background 카드에 맞게 text를 띄우려고 한다.
- 그래서 알아본 결과, 캔버스 내의 object의 경우, 좌표값을 받아 띄우는 lineCoords 속성이 있다.
- text가 만들어졌는지 알기 위해 flag 변수를 만든다.

```js
let isTextCreated = false;

let defaultOptions = {
  lockScalingX: true,
  lockScalingY: true,
  lockRotation: true,
  lockMovement: true,
}

if (this.canvas) {
  if (!isTextCreated) {
    const cardobj = this.canvas.getObjects().filter((item) => item.name.includes("background"));
    const bottomTextMargin = 7;
    const sideMargin = 9;
    const bottomFontSize = 7.5;

    const leftTextBox = new fabric.TextBox('', {
      fontSize : bottomFontSize,
      fill: 'black',
      fonFamilty: 'Malgun Gothic',
      name: _.uniqueId("text_left"), //"text_left1"
      width: 50,
      left: cardObj[0].lineCoords.bl.x + 9,
      top: cardObj[0].lineCoords.bl.y,
      height: 100,
      textAlign: 'left',
      ...defaultOptions
    })
    .
    .
    //텍스트 생성
    isTextCreated = !isTextCreated 
  } else {
    //텍스트 제거
    const textArray = this.canvas.getObjects().find((item) => item.name.includes("text_"));
    for (const element of textArray) {
      this.canvas.remove(element);
    }

    isTextCreated = !isTextCreated;
  }
}
```

- 하지만 위처럼 left 속성에 7을 더하면 나중에 값을 바꿀 때 실수할 여지가 높다.
- 따라서 변수로 만들어서 관리한다. 매직넘버를 만들지 않는다는 의미다.
- sideMargin만큼 왼쪽에서 띄워주기 위해 sideMargin을 주는 것이다.

```js
const sideMargin = 9;
const leftTextBox = new fabric.TextBox('', {
  fontSize : bottomFontSize,
  fill: 'black',
  fonFamilty: 'Malgun Gothic',
  name: _.uniqueId("text_left"), //"text_left1"
  width: 50,
  left: cardObj[0].lineCoords.bl.x + sideMargin,
  top: cardObj[0].lineCoords.bl.y,
  height: 100,
  textAlign: 'left',
  ...defaultOptions
})
```

- 하지만 위처럼 주게 되면, sideMargin이 눈에 띄지 않는다.
- 따라서 따로 setter를 만들어서 뺐다.
- 줄을 바꿔서 해주니 주석을 달아주면 더 눈에 띈다.

```js
const sideMargin = 9;
const leftTextBox = new fabric.TextBox('', {
  fontSize : bottomFontSize,
  fill: 'black',
  fonFamilty: 'Malgun Gothic',
  name: _.uniqueId("text_left"), //"text_left1"
  width: 50,
  left: cardObj[0].lineCoords.bl.x,
  top: cardObj[0].lineCoords.bl.y,
  height: 100,
  textAlign: 'left',
  ...defaultOptions
})
leftTextBox.set('left', leftTextBox.left + ( sideMargin )); //sideMargin만큼 띄워준다.
```

- leftTextBox는 9만큼의 margin을 주어 왼쪽에서 9 pixel만큼 띄웠다.
- 똑같은 방식으로 top을 주려했는데, 원하는대로 작동하지 않았다. 


```js
const bottomTextMargin = 7;
leftTextBox.set('top', leftTextBox.top - ( bottomTextMargin )); //sideMargin만큼 띄워준다.
```

- 이유는 fontSize였다. fontSize만큼 top값에서 빼줘야 한다.
- 이 때 setter를 따로 뺀 효과를 발휘한다. 만약 객체를 생성하는 안에서 top값에서 fontSize를 빼려했다면?
- 아직 initialize 되지 않았기 때문에 오류가 났을 것이다.

```js
const bottomTextMargin = 7;
leftTextBox.set('top', leftTextBox.top - ( bottomTextMargin + leftTextBox.fontSize)); //sideMargin만큼 띄워준다.
```


- 오른쪽 텍스트는 오른쪽 정렬 기준으로 만들어달라는 요청이 있었다.
- 오른쪽 텍스트도 왼쪽처럼 하려 했지만, 왼쪽처럼 하면 오류가 난다.
- 오른쪽 텍스트는 lineCoords 중 br 속성값을 가져와야 한다.
- 오른쪽 텍스트는 오른쪽 정렬이라 left에서 sideMargin만이 아니라 width만큼도 빼줘야 한다.
- 여기서도 객체 생성 중에 width 값을 가져오기는 어렵다. setter를 따로 뺀게 장점을 발휘한다.
- top은 이전과 동일하게 폰트사이즈와 원하는 마진값 만큼을 빼준다.

```js
const rightTextBox = new fabric.TextBox('', {
  fontSize: bottomFontSize,
  fill: 'black',
  fontFamily: 'Malgun Gothic',
  name: _.uniqueId("text_right"),
  width: 83,
  left: cardObj[0].lineCoords.br.x,
  top: cardObj[0].lineCoords.br.y,
  height: 100,
  textAlign: 'right',
  ...defaultOptions
})
rightTextBox.set('left', rightTextBox.left - (sideMargin + rightTextBox.width));
rightTextBox.set('top', rightTextBox.top - (bottomTextMargin + rightTextBox.fontSize));
```

- 센터 텍스트는 left와 top 값 계산이 좀 다르다. 가운데 정렬이기 때문이다.
- bl의 x와 br의 x를 더해 2로 나눈, 즉 중점이 바로 텍스트의 left 값(시작점)이 된다.
- width는 카드의 넓이 만큼으로 먼저 정한다.
- 하지만 실제로는 양 옆이 모두 9 pixel 만큼 띄워야 하므로, 양옆의 sideMargin만큼을 width에서 빼준다.
- left 값은 가운데 정렬이기 때문에 width의 절반만큼을 left 값에서 빼준다.
- top은 이전과 다르게 띄워야하는 만큼을 전부 띄워준다. 5와 5.5는 줄 사이의 공간이다.

```js
const centerTextBox = new fabric.TextBox('', {
  fontSize: 15,
  fill: 'black',
  fontFamily: 'Malgun Gothic',
  name: _.uniqueId("text_center"),
  width: cardObj[0].lineCoords.br.x - cardObj[0].lineCoords.bl.x,
  left: cardObj[0].lineCoords.br.x,
  top: cardObj[0].lineCoords.br.y,
  height: 100,
  textAlign: 'center',
  ...defaultOptions
})
rightTextBox.set('width', rightTextBox.width - 2 * sideMargin );
rightTextBox.set('left', rightTextBox.left - ( rightTextBox.width ));
rightTextBox.set('top', rightTextBox.top - (bottomFontSize + leftText.fontSize + 5 + lineBorderSize + 5.5  + rightTextBox.fontSize));
```


## <span style="color:#802548">_모바일 웹, 모바일 앱을 모두 고려한 image 가져오기_</span>
- 모바일 웹은 input type file으로 충분하지만, 모바일 앱은 input type file이 아니라 bridge 함수를 써야한다.
- 모바일 앱은 갤러리에 접근하기 위해서 앨범 접근 권한 검사등을 수행해야 하기 때문이다.
- 그런데 bridge함수가 아래처럼 구성되어 있었다. 비동기라는 의미다.


```js
function send(method, parameter, callBack) {
    if (deviceInfo.name !=='shinhancard') return;
    .
    .
    .
    setTimeout(() => {
        WebNativeBridge.callHandler = callBack();
    }, 10)
}
```

- 비동기였기 때문에, 진행 할 코드가 존재한다면 진행할 코드가 모두 진행된 뒤에 진행되는 형태였다.
- 따라서 send를 아래와 같이 사용하게 되면 file의 값이 채워지기 전에 아래 코드들이 진행된다. 
- 따라서 app에서 실행되는 경우, file은 undefined로만 값이 나오게 된다.

```js
const imageUpload = () => {
    let file = null;
    // 레이어 개수 10개 초과되면 추가 안되도록 설정
    if (this.canvas.getObjects().length > 11) {
        // alert('관리 가능한 레이어 최대 개수는 10개 입니다.');
        openModal("modal04");
        return;
    }
    if (deviceInfo.name === 'shinhancard') {
        const param = {
                        header: 'getPhoto',
                        from: 'album'
                    };
        send('shop_command', param, function(data) {
            if(data typeof String) {
                data = String(JSON.parse(data));
            }

            if (data.resultCode === '0') {
                data.imageData = data.imageData.replace(/^chekrtttrt==/, '');
                file = data.imageData;
            } else if (data.resultCode === '1013') {

            } else if (data.resultCode === '1014') {

            }
        });
    } else {
        file = event.target.files[0]; // 선택한 파일 가져오기
    }
    

    /* 앱 실행의 경우 사실상 아래부터 전부 오류 */
    if (!file) return; // 파일이 없으면 종료
    
    // 파일 확장자 제한
    const fileName = file.name.toLowerCase()
    const extension = fileName.split('.').pop().toLowerCase();
    if (['jpg', 'jpeg', 'png', 'bmp', 'gif'].indexOf(extension) === -1) {
        openModal("modal05");
        return;
    }

    const isMeta = await checkMetaData(file);

    if(fileName.includes('스크린샷') || fileName.includes('screenshot') || !isMeta) {   
        openModal("modal10")
        return;
    }

    $loadingBox.style.display = "block";
    const reader = new FileReader();
    reader.onload = async (event) => {
        // 강아지,고양이 판별
        let imgObj = new Image();
        imgObj.src = event.target.result;

        imgObj.onload = async () => {
            const pet = await classifyImage(imgObj);
            
            if (undefined !== petType.find((type) => type === pet)) {
                const imageTextureSize = imgObj.width > imgObj.height ? imgObj.width : imgObj.height;

                if (imageTextureSize > fabric.textureSize) {
                        fabric.textureSize = imageTextureSize;
                }
                // Fabric.js Image 객체 생성
                const fabricImage = new fabric.Image(imgObj, {
                    borderOpacityWhenMoving: 1,
                    name: _.uniqueId("image_"),
                    // 이미지보정 작업 시 필요한 필터객체 추가
                    filters: [
                        new fabric.Image.filters.BaseFilter(),
                        new fabric.Image.filters.Brightness({brightness:0}),
                        new fabric.Image.filters.Contrast({contrast:0}),
                        new fabric.Image.filters.Saturation({saturation:0}),
                        new fabric.Image.filters.HueRotation({rotation:0}),
                    ],
                });

                // 캔버스의 크기
                const canvasWidth = this.canvas.width;
                const canvasHeight = this.canvas.height;

                // 이미지의 크기
                const imgWidth = fabricImage.width;
                const imgHeight = fabricImage.height;

                // 축소 비율 계산
                const widthRatio = (canvasWidth - 100) / imgWidth;
                const heightRatio = (canvasHeight - 100) / imgHeight;

                // 비율 중 작은 값을 선택
                const scale = Math.min(widthRatio, heightRatio);

                // 이미지가 캔버스를 초과하는 경우에만 축소
                if (scale < 1) {
                    fabricImage.scale(scale);
                }

                fabricImage.origin = imgObj;

                
                // 이미지가 있을 경우 교체
                const bfImage = this.canvas.getObjects().find((item) => item.name.includes("image_"));
                if (bfImage) {
                    this.canvas.remove(bfImage);
                    this.canvas.fire("object:remove", {target : bfImage});
                    this.canvas.requestRenderAll();
                }
                
                // 이미지 중앙 배치
                this.canvas.centerObject(fabricImage);
                // Canvas에 이미지 추가
                this.canvas.add(fabricImage);
            } else {
                openModal("modal01")
                //alert("강아지, 고양이 사진만 가능합니다");
            }
            
            $loadingBox.style.display = "none";
        };
    };
    reader.readAsDataURL(event.target.files[0]);
}

$fi.onchange = imageUpload;
```

- 해당 상황을 해결하려면 똑같은 함수를 두 번 호출해야 한다.
- if문과 else문에서 똑같은 로직을 진행하고, 그 이후 진행하는 코드가 하나도 없으면 된다는 의미다.
- 하지만 이는 코드 낭비이므로, function으로 빼준다.
- 맨끝의 event.target.files(0)을 file로 바꿔준다.
- 또한 this binding을 해준다.

```js
function addImageOnCanvas(file) {
    /* 앱 실행의 경우 사실상 아래부터 전부 오류 */
    if (!file) return; // 파일이 없으면 종료
    
    // 파일 확장자 제한
    const fileName = file.name.toLowerCase()
    const extension = fileName.split('.').pop().toLowerCase();
    if (['jpg', 'jpeg', 'png', 'bmp', 'gif'].indexOf(extension) === -1) {
        openModal("modal05");
        return;
    }

    const isMeta = await checkMetaData(file);

    if(fileName.includes('스크린샷') || fileName.includes('screenshot') || !isMeta) {   
        openModal("modal10")
        return;
    }

    $loadingBox.style.display = "block";
    const reader = new FileReader();
    reader.onload = async (event) => {
        // 강아지,고양이 판별
        let imgObj = new Image();
        imgObj.src = event.target.result;

        imgObj.onload = async () => {
            const pet = await classifyImage(imgObj);
            
            if (undefined !== petType.find((type) => type === pet)) {
                const imageTextureSize = imgObj.width > imgObj.height ? imgObj.width : imgObj.height;

                if (imageTextureSize > fabric.textureSize) {
                        fabric.textureSize = imageTextureSize;
                }
                // Fabric.js Image 객체 생성
                const fabricImage = new fabric.Image(imgObj, {
                    borderOpacityWhenMoving: 1,
                    name: _.uniqueId("image_"),
                    // 이미지보정 작업 시 필요한 필터객체 추가
                    filters: [
                        new fabric.Image.filters.BaseFilter(),
                        new fabric.Image.filters.Brightness({brightness:0}),
                        new fabric.Image.filters.Contrast({contrast:0}),
                        new fabric.Image.filters.Saturation({saturation:0}),
                        new fabric.Image.filters.HueRotation({rotation:0}),
                    ],
                });

                // 캔버스의 크기
                const canvasWidth = this.canvas.width;
                const canvasHeight = this.canvas.height;

                // 이미지의 크기
                const imgWidth = fabricImage.width;
                const imgHeight = fabricImage.height;

                // 축소 비율 계산
                const widthRatio = (canvasWidth - 100) / imgWidth;
                const heightRatio = (canvasHeight - 100) / imgHeight;

                // 비율 중 작은 값을 선택
                const scale = Math.min(widthRatio, heightRatio);

                // 이미지가 캔버스를 초과하는 경우에만 축소
                if (scale < 1) {
                    fabricImage.scale(scale);
                }

                fabricImage.origin = imgObj;

                
                // 이미지가 있을 경우 교체
                const bfImage = this.canvas.getObjects().find((item) => item.name.includes("image_"));
                if (bfImage) {
                    this.canvas.remove(bfImage);
                    this.canvas.fire("object:remove", {target : bfImage});
                    this.canvas.requestRenderAll();
                }
                
                // 이미지 중앙 배치
                this.canvas.centerObject(fabricImage);
                // Canvas에 이미지 추가
                this.canvas.add(fabricImage);
            } else {
                openModal("modal01")
                //alert("강아지, 고양이 사진만 가능합니다");
            }
            
            $loadingBox.style.display = "none";
        };
    };
    reader.readAsDataURL(file);
}
addImageOnCanvas = addImageOnCanvas.bind(this);
```

- 길던 business logic을 전부 빼서 else문 혹은 if문 안에서 함수의 진행이 모두 종료된다.
- 따라서 setTimeout에 따라 event loop에 들어갔던 send()가 call stack으로 올라온다.
- call stack으로 올라와서 bridge를 호출하고 sucessCallback에 따라 진행된다.
- successCallback에서 resultCode가 0, 즉 앨범에서 이미지를 선택했다면 imageData를 가져와서 file을 만들어서 canvas에 띄운다.

```js
const imageUpload = () => {
    let file = null;
    // 레이어 개수 10개 초과되면 추가 안되도록 설정
    if (this.canvas.getObjects().length > 11) {
        // alert('관리 가능한 레이어 최대 개수는 10개 입니다.');
        openModal("modal04");
        return;
    }
    if (deviceInfo.name === 'shinhancard') {
        const param = {
                        header: 'getPhoto',
                        from: 'album'
                    };
        send('shop_command', param, function(data) {
            if(data typeof String) {
                data = String(JSON.parse(data));
            }

            if (data.resultCode === '0') {
                data.imageData = data.imageData.replace(/^chekrtttrt==/, '');
                // 바이너리 데이터 얻기
                const binaryData = base64ToBinary(data.imageData);
                // 바이너리 데이터를 사용하여 Blob 생성
                const blob = new Blob([binaryData], { type: 'image/png' });
                // Blob을 사용하여 File 객체 생성
                const file = new File([blob], 'image.png', { type: 'image/png' });
                addImageOnCanvas(file);
            } else if (data.resultCode === '1013') {
                alert(data.resultMessgae);
            } else if (data.resultCode === '1014') {
                alert(data.resultMessgae);
            }
        });
    } else {
        file = event.target.files[0]; // 선택한 파일 가져오기
        addImageOnCanvas(file);
    }
}

function base64ToBinary(base64) {
    const binaryString = atob(base64);
    const len = binaryString.length;
    const bytes = new Uint8Array(len);
    for (let i = 0; i < len; i++) {
        bytes[i] = binaryString.charCodeAt(i);
    }
    return bytes;
}

$fi.onchange = imageUpload;
```

- 근데 보통 테스트계는 소스를 올리는 과정이 매우 귀찮으므로 에러를 바로 알수 있게 처리해준다.
- addImageOnCanvas()에서 일어난 error가 상위 call stack인 imageUpload로 올라와서 처리된다.
- 따라서 addImageOnCanvas에서 일어난 에러도 alert가 된다. 물론 data를 replace하다 나는 에러도 alert가 뜬다.

```js
const uploadImage = () => {
    let file = null;
    // 레이어 개수 10개 초과되면 추가 안되도록 설정
    if (this.canvas.getObjects().length > 11) {
        // alert('관리 가능한 레이어 최대 개수는 10개 입니다.');
        openModal("modal04");
        return;
    }
    if (deviceInfo.name === 'shinhancard') {
        const param = {
                        header: 'getPhoto',
                        from: 'album'
                    };
        send('shop_command', param, function(data) {
            if(data typeof String) {
                data = String(JSON.parse(data));
            }

            if (data.resultCode === '0') {

                try {
                    data.imageData = data.imageData.replace(/^chekrtttrt==/, '');
                    // 바이너리 데이터 얻기
                    const binaryData = base64ToBinary(data.imageData);
                    // 바이너리 데이터를 사용하여 Blob 생성
                    const blob = new Blob([binaryData], { type: 'image/png' });
                    // Blob을 사용하여 File 객체 생성
                    const file = new File([blob], 'image.png', { type: 'image/png' });
                    addImageOnCanvas(file);
                } catch (error) {
                    alert(error);
                }
                
            } else if (data.resultCode === '1013') {
                alert(data.resultMessgae);
            } else if (data.resultCode === '1014') {
                alert(data.resultMessgae);
            }
        });
    } else {
        file = event.target.files[0]; // 선택한 파일 가져오기
        addImageOnCanvas(file);
    }
}
$fi.onchange = imageUpload;

function base64ToBinary(base64) {
    const binaryString = atob(base64);
    const len = binaryString.length;
    const bytes = new Uint8Array(len);
    for (let i = 0; i < len; i++) {
        bytes[i] = binaryString.charCodeAt(i);
    }
    return bytes;
}
```

