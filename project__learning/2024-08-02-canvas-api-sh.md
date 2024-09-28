## <span style="color:#802548">_this keyword_</span>

- 화살표 함수를 안 쓰고 function keyword를 쓴 뒤 this를 호출하면 this가 제대로 인식되지 않았다.

```js
this.canvas.on("mouse:down", (event) =>  {
  if (event.target) {
    const clickedObject = event.target;

    // 배열에 클릭된 객체 추가 (최신 클릭이 맨 뒤에 오도록)
    const index = clickedObjects.indexOf(clickedObject);
    if (index !== -1) {
      clickedObjects.splice(index, 1); // 이미 있는 경우 제거 후
    }
    clickedObjects.push(clickedObject); // 맨 뒤에 추가

    // 최근 클릭 순서대로 앞으로 가게 변경
    changeStickerDisplayOrder(this.canvas); //this가 제대로 인식되지 않음.
  }
});

function changeStickerDisplayOrder(canvas) {
  clickedObjects.forEach(function (obj) {
    canvas.bringToFront(obj);
  });
}
```

- function keyword를 쓴 경우, this가 window를 가리키기 때문이다.
- function keyword에서 this를 쓰고 싶다면, 아래같이 function 밖에서 this를 받아와서 써야 한다. 

```js
const self = this;
function checkAmount(textBox){
    const maxLength = 7;
    if(textBox.text.length > maxLength ) {
        textBox.set('text',textBox.text.slice(0,maxLength));
        self.canvas.discardActiveObject();
        self.canvas.setActiveObject(textBox);
        self.canvas.requestRenderAll();
    }
}
```

- 화살표 함수에서 상위 context에 this를 binding하는 것과 완전하게 동일하다. 다만 소스코드만 다를 뿐이다.

```js
const checkAmount = (textBox) => {
  const maxLength = 7;
  if(textBox.text.length > maxLength ) {
      textBox.set('text',textBox.text.slice(0,maxLength));
      self.canvas.discardActiveObject();
      self.canvas.setActiveObject(textBox);
      self.canvas.requestRenderAll();
  }
}
```


- this는 eventHandler에서 쓰이면 다른 의미로 해석된다.
- onchange나 addEventListener에서 this를 사용하게 되면 해당 element를 this로 가리키게 된다.
- 따라서 거기서 this를 사용하려면 아래처럼은 사용할 수 없다.

```js
els.fontColor.forEach(color => { // 폰트 색상
    color.onchange = (event) => {
      const value = event.target.value;
      setFont({fill : value});
    }
    this.canvas.getObjects(). //this는 dom이고, dom에는 canvas가 없음. 따라서 undefined
});
```


- 아래처럼 사용해야 한다.

```js
const self = this;
els.fontColor.forEach(color => { // 폰트 색상
    color.onchange = (event) => {
      const value = event.target.value;
      setFont({fill : value});
    }
    self.canvas.getObjects().
});
```

```js
 const self = this;
    this.canvas.on("object:scaling", function() {
       const imageObj = self.canvas.getObjects().find((obj) => obj.name?.includes('image'));
        const cropObj = self.canvas.getObjects().find((obj) => obj.name?.includes('cropZone'));
    })
```

- 위처럼 사용해도 this 자체가 undefined인 경우가 있다.
- 그럴 때는 this가 제대로 binding되었는지 확인이 필요하다.
- 아래를 보면 initFont로 들어올 때, this를 제대로 바인딩하지 않은 것을 확인할 수 있다.
  - call, bind, apply 함수가 없다. 그럼 this가 binding이 이뤄지지 않는다.
- 그 어떠한 this binding이 없다. 따라서 initFont에서 호출하는 this는 모두 undeinfed가 된다.

```js
const initText = function () {
        
    const init = () => {
      let els = {
            fontTypeSel: getEl('#fontTypeSel'),
            fontSizeSel: getEl('#fontSizeSel'),
            fontStyle: getEl('#fontStyleSel'),
            fontColor: getElByName('fontColor'),
            fontAlign: getElByName('fontAlign')
        }
      initFont(els,this.canvas);
    }
}

const initFont = function() {
  .
  .
  .
  const self = this; //undefined
  els.fontColor.forEach(color => { // 폰트 색상
    color.onchange = (event) => {
      const value = event.target.value;
      setFont({fill : value});
    }
    self.canvas.getObjects().
  });
  .
  .
}
```

- 이런 경우에는 this를 사용하지 않고 똑같이 param으로 넘기는 편이 수월하다.

```js
initFont(els,this.canvas, this.foregroundCanvas);

const initFont = (else, canvas, foregroundCanvas) => {

}
```


- this를 제대로 binding하는 방식으로 진행하고 싶을 수도 있다.
- 이는 3번에서 알아보자.


## <span style="color:#802548">_생성자 함수에서 new 가 필수인 이유_</span>

- 아래에서 new를 지우면 모든 js가 제대로 출력되지 않는다.

```js
const goEditor = async () => {
    const editorHtml = await getHtmlView(editorPath);
    const loadingHtml = await getHtmlView(loadingPath);
    const modalHtml = await getHtmlView(modalPath);

    $app.innerHTML = await editorHtml;
    $app.innerHTML += await loadingHtml;
    $app.innerHTML += await modalHtml;

    // Ensure PhotoEditor is called with `new`
    new PhotoEditor('canvas');
}
```

- 더 구체적으론 아래와 같은 오류가 뜬다.

```
Uncaught (in promise) TypeError: Cannot set properties of undefined (setting 'canvas')
    at PhotoEditor (core.js:36:17)
    at goEditor (router.js:51:9)
```


- 그 이유는 this가 setting이 안되었기 때문이다.
- new 없이는 this가 해당 photoEditor를 가리키는 지 알수가 없기 때문에 this 자체를 알 수 없는 것이다.

```js
this.canvas = new fabric.Canvas(canvasId, {
        isDrawingMode: false,
        width: canvasWrap.clientWidth - 261,
        height: canvasWrap.clientHeight,
    });
```

- 다만 그 경우, function constructor 방식으로 instance를 만든 것이다.
- class constructor 방식으로 만든 게 아니라 가독성이 떨어진다. 엔간하면 class로 만들자.
- 아래에서 this.canvas는 사실 PhtoEditor prototype의 instance에 canvas라는 field를 setting한 것이다!

```js
const PhotoEditor = function (canvasId) {
    const canvasWrap = getEl(".canvas-wrap")

    this.canvas = new fabric.Canvas(canvasId, {
        isDrawingMode: false,
        width: canvasWrap.clientWidth - 261,
        height: canvasWrap.clientHeight,
    });

    this.foregroundCanvas = new fabric.Canvas('foreground-canvas', {
        isDrawingMode: false,
        width: canvasWrap.clientWidth - 261,
        height: canvasWrap.clientHeight,
    });


    this.activeSelection = null;
    this.canvas.tool = TOOL.move;

    const htmlPath = {
        header: "/src/views/header.html",
        photoHeader: "/src/views/photoHeader.html",
        layer: "/src/views/layerPanel.html",
        size: "/src/views/properties/sizePanel.html",
        brush: "/src/views/properties/brushPanel.html",
        erase: "/src/views/properties/erasePanel.html",
        text: "/src/views/properties/textPanel.html",
        sticker: "/src/views/properties/stickerPanel.html",
        retouch: "/src/views/properties/retouchPanel.html",
        filter: "/src/views/properties/filterPanel.html",
        color: "/src/views/properties/colorPanel.html",
        removebg:"/src/views/properties/removePanel.html",
    };
    const divId = {
        header: "header",
        layer: "layerPanel",
        properties: "propertiesPanel",
    };

    window.addEventListener("beforeunload", function (event) {
        console.log("reload");
    });

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
        this._initFont = initFont.call(this);
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

    const viewInit = () => {
        return new Promise(async (resolve, reject) => {
            let headerInit;
            if(window.location.pathname == '/photoEditor') {
                headerInit = await initView(htmlPath.photoHeader, divId.header);
            } else {
                headerInit = await initView(htmlPath.header, divId.header);
            }
            const layerInit = await initView(htmlPath.layer, divId.layer);

            // 속성 패널 html 삽입
            const sizeInit = await initView(htmlPath.size, divId.properties);
            const brushInit = await initView(htmlPath.brush, divId.properties);
            const textInit = await initView(htmlPath.text, divId.properties);
            const stickerInit = await initView(
                htmlPath.sticker,
                divId.properties
            );
            const retouchInit = await initView(
                htmlPath.retouch,
                divId.properties
            );
            const filterInit = await initView(
                htmlPath.filter,
                divId.properties
            );
            const colorInit = await initView(htmlPath.color, divId.properties);
            const eraseInit = await initView(htmlPath.erase, divId.properties);
            const removeBgInit = await initView(htmlPath.removebg, divId.properties);
            if (
                headerInit &&
                layerInit &&
                sizeInit &&
                brushInit &&
                textInit &&
                stickerInit &&
                retouchInit &&
                filterInit &&
                colorInit &&
                eraseInit && 
                removeBgInit
            ) {
                accordion();
                modal();
                swiper();
                stickerSwiper();
                activeBtn();
                resolve(true);
            } else reject();
        });
    };

    viewInit()
        .then(() => {
            init();
        })
        .catch((error) => {
            console.error("init HTML을 불러오는 동안 오류 발생:", error);
        });

    return this;
};
```


## <span style="color:#802548">_call, apply_</span> 

- 다만 위의 this.canvas는 전역변수로서 조금 특수한 양식을 보인다.
- initFont는 

```js
import initFont from './initFont';

const initText = function () {
  const init = () => {
    els = {
        fontTypeSel: getEl('#fontTypeSel'),
        fontSizeSel: getEl('#fontSizeSel'),
        fontStyle: getEl('#fontStyleSel'),
        fontColor: getElByName('fontColor'),
        fontAlign: getElByName('fontAlign')
    }
    initFont(els, this.canvas);
  }

  init();
}
```

- 본래의 init은 아래와 같은 형태인데, initFont는 initText 안에서 이뤄진다.
- initFont를 내가 지금은 추가시켜놓은 상황이다.

```js
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
    this._initFont = initFont.call(this);
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

- 사실 _initCanvas로 this에 추가할 필요도 없다.
- 아래와 같이 해도 똑같이 잘 작동한다.
- 다만 여기서 call을 제외시키면 제대로 작동하지 않는다.

```js
const init = () => {
    initCanvas.call(this);
    InitImageUpload.call(this);
    InitActiveSelection.call(this);
    ControllViewport.call(this);
    Preview.call(this);
    initUndo.call(this);
    InitLayer.call(this);
    InitCrop.call(this);
    initText.call(this);
    initFont.call(this);
    initIcon.call(this);
    initRetouch.call(this);
    initResize.call(this);
    initFilter.call(this);
    //initBrush.call(this);
    initFlip.call(this);
    // initMove.call(this);
    initReset.call(this);
    Confirmation.call(this);
    initErase.call(this);
    initRotate.call(this);

    // 에디터 일 경우만 배경 제거, 배경 색상 변경 호출
    if (window.location.pathname == "/diyEditor") {
      this._backgroundRemove = backgroundRemove.call(this);
      this._initColor = initColor.call(this);
    }
  };
```

- 해당 맥락에서 아까의 this keyword를 살펴보자.
- this keyword가 안되었던 이유는 아래와 같다.

```js
const self = this;
els.fontColor.forEach(color => { // 폰트 색상
    color.onchange = (event) => {
      const value = event.target.value;
      setFont({fill : value});
    }
    self.canvas.getObjects().
});
```

- 아래에서 initFont를 할 때, this keyword가 어떤 대상을 가리킬 지 정하지 않았다.
- 따라서 반드시 this가 undeinfed일 수밖에 없었다.

```js
const initText = function () {
  const init = () => {
    els = {
        fontTypeSel: getEl('#fontTypeSel'),
        fontSizeSel: getEl('#fontSizeSel'),
        fontStyle: getEl('#fontStyleSel'),
        fontColor: getElByName('fontColor'),
        fontAlign: getElByName('fontAlign')
    }
    initFont(els, this.canvas);
  }

  init();
}
```

- 해당 상황의 경우, this를 어떤 것으로 binding할 지 결정해줘야 한다.
- 당연히 photoeditor의 this다.
- photoEditor의 this를 어떻게 가져와야 할까? 고민할 필요 없다.
- 어차피 우리는 initText의 this context를 PhotoEditor로 만들어놓았다.
- 아래 call의 this는 PhtoEditor 자신의 instance를 의미한다.

```js
const PhotoEditor = function (canvasId) {
  const init = () => {
    .
    .
    
      initText.call(this);
      initFont.call(this);
    .
    .
    .
      initIcon.call(this);
      initRotate.call(this);

      // 에디터 일 경우만 배경 제거, 배경 색상 변경 호출
      if (window.location.pathname == "/diyEditor") {
        this._backgroundRemove = backgroundRemove.call(this);
        this._initColor = initColor.call(this);
      }
  };

  .
  .
  .
  
  viewInit()
        .then(() => {
            init();
        })
        .catch((error) => {
            console.error("init HTML을 불러오는 동안 오류 발생:", error);
        });

    return this; //그냥 return도 되고, return this도 되고, 그냥 없어도 됨. 만들어진 instance를 return함.
}
```


- initText의 this는 이제 무조건 photoEditor instance다.
- 더 구체적으로는 canvasId가 canvas인 instance가 this가 된다.

```js
const goEditor = async() => {
    const editorHtml = await getHtmlView(editorPath);
    const loadingHtml = await getHtmlView(loadingPath);
    const modalHtml = await getHtmlView(modalPath);
    
    $app.innerHTML = await editorHtml;
    $app.innerHTML += await loadingHtml;
    $app.innerHTML += await modalHtml;
    
    new PhotoEditor('canvas');
}
```

- 따라서 initText 안에서 this를 호출해도 photoEditor instance가 된다.

```js
const initText = function () {
    const tool = TOOL;
    const $btnText = getEl('#btnText');
    let els = {}
    let fontStyleObj = { 
        fontSize: 16,
        fill: 'black',
        width: 100,  // 임시 처리 -- 레이어 영역 수정 필요(퍼블)
        height: 100,
        fontFamily : 'Malgun Gothic'  // 기본 글꼴 지정 -- 임시처리
    };
        
    const init = () => {
        els = {
            fontTypeSel: getEl('#fontTypeSel'),
            fontSizeSel: getEl('#fontSizeSel'),
            fontStyle: getEl('#fontStyleSel'),
            fontColor: getElByName('fontColor'),
            fontAlign: getElByName('fontAlign')
        }
        console.log(this);
        initFont(els,this.canvas);
    }
}
```

- 그러나 initFont는 this의 context를 정하지 않아 undefined다.
- 만약 initFont가 parameter가 없었다면 그냥 동일하게 call을 사용하면 된다.

```js
initFont.call(this);
```

- 혹시 bind를 사용했다면 아래처럼 써줘야 한다.
- bind 함수는 함수를 발동시키지 않기 때문이다.

```js
initFont.bind(this);
initFont();
```


- 하지만 initFont는 parameter가 있다.

```js
const initFont = function (els, canvas) {
.
.
.

  fontInit();
}

export default initFont
```

- 따라서 apply를 활용해야 한다. 여기서 this는 initText가 갖는 this다.
- 그건 이전에 말했듯 canvas라는 canvasId를 가진 PhtoEditor instance다.
- parameter로 들어갈 것들은 모두 배열에 넣어준다.

```js
const initText = function () {
        
    const init = () => {
      let els = {
            fontTypeSel: getEl('#fontTypeSel'),
            fontSizeSel: getEl('#fontSizeSel'),
            fontStyle: getEl('#fontStyleSel'),
            fontColor: getElByName('fontColor'),
            fontAlign: getElByName('fontAlign')
        }
      initFont.apply(this,[els,this.canvas]);
    }
}
```

- this binding할 함수에 parameter가 없다면 call이다.
- this binding할 함수에 parameter가 있다면 apply이다.
- this binding을 하고 동시에 함수가 발동하는 것을 원하지 않는다면 bind다.



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


- script import를 가져올 때, export나 import가 없는 library는 src로 가져온다.
- library를 가져오는 순서도 중요하다. coco-ssd는 tf.js를 필요로 하기에, tf.js를 먼저 가져와야 한다.

```html
<script src="/conts/photo/libjs/fabric.js"></script>
<script src="/conts/photo/libjs/tf.js.js"></script>
<script src="/conts/photo/libjs/coco-ssd.js"></script>
<script src="/conts/photo/libjs/lodash.js"></script>
```

- tf.js나 coco-ssd.js에 require 문법이 있을 수 있다.
- 하지만 node modules가 없어도 require를 읽어온다. 따라서 문제가 없었다.
- 그 이유는 libray js 내에서 node 환경인지, AMD 환경인지, browser환경(대부분의 내부망-외부인터넷 연결X, 노드X, bundler x- 쓰는 legacy들)인지 검사하기 때문이다.
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

## <span style="color:#802548">_tensorflow model caching하기_</span>

- google cloud model을 사용하는 tensorflow의 경우, 모델을 처음에 가져와야 판독이 가능하다.
- 따라서 모델을 가져오는 코드를 만들었었다.

```js
const classifyImage = async (imgObj) => {

    // cocossd 모델 로드
    const model = await cocoSsd.load();
    
    // 이미지를 텐서로 변환
    const tensor = tf.browser.fromPixels(imgObj);

    // 모델에 이미지 전달하여 예측
    const predictions = await model.detect(imgObj);

    // 예측 결과 출력
    console.log(predictions);

    // 예측 결과 중 가장 높은 확률의 라벨 가져오기
    const topPrediction = predictions[0]?.class;

    // 결과를 화면에 표시
    // document.getElementById('petTypeText').innerText = `이것은 ${topPrediction} 입니다.`;

    return topPrediction;

}

export default classifyImage
```

- 그런데 문제는 모델을 가져오는 행위가 이미지 판독마다 반복되면서, 모델이 쌓여 메모리 leak이 일어났다.
- 따라서 캐싱이 필요해졌다. 따라서 model변수를 아래처럼 캐싱해서 사용한다.
- 모델변수는 해당 함수밖에 존재하기 때문에, 한번만 불러오면 그 뒤로는 로드하지 않는다.

```js
let loadedModel = null;

const classifyImage = async (imgObj) => {
    // cocossd 모델 로드
    if (!loadedModel) {
        loadedModel = await cocoSsd.load();
    }
    const model = loadedModel;
    // const model = await cocoSsd.load();

    // 이미지를 텐서로 변환
    const tensor = tf.browser.fromPixels(imgObj);

    // 모델에 이미지 전달하여 예측
    const predictions = await model.detect(imgObj);

    // 예측 결과 출력
    console.log(predictions);

    // 예측 결과 중 가장 높은 확률의 라벨 가져오기
    const topPrediction = predictions[0]?.class;

    // 결과를 화면에 표시
    // document.getElementById('petTypeText').innerText = `이것은 ${topPrediction} 입니다.`;

    return topPrediction;
};
```



## <span style="color:#802548">_tensorflow model 외부통신 없이 model 사용하기_</span>

- 그런데 legacy 내부망 환경에서는 model을 외부와 통신하여 사용하는 것이 불가능했다.
- 따라서 model을 직접 안으로 들여와서 사용해야 했다.
- 따라서 아래처럼 model을 내부에서 가져오는 방식으로 변경했다.

```js
let loadedModel = null;

const classifyImage = async (imgObj) => {
    // cocossd 모델 로드
  if (!loadedModel) {
      // loadedModel = await cocoSsd.load();
      loadedModel = await cocoSsd.load({
              base: 'lite_mobilenet_v2',
              modelUrl: 'src/assets/models/models.json'
          }
      );
  }
  .
  .
  .
  
  return topPrediction;
};
```

- load를 위와 같이 한 이유는 아래가 공식 refer의 선언이기 때문이다.

```js
//https://github.com/tensorflow/tfjs-models/tree/master/coco-ssd
export interface ModelConfig {
  base?: ObjectDetectionBaseModel;
  modelUrl?: string;
}

cocoSsd.load(config: ModelConfig = {});
```

- models.json과 그에 필요한 base 들은 구글에서 다운로드 가능하다.

```
https://www.tensorflow.org/js/guide/conversion?hl=ko 에서 TensorFlow Hub 모듈: 모델을 공유하고 찾기 위한 플랫폼인 TensorFlow Hub에서 배포하기 위해 패키지로 구성되는 모델입니다. 모델 라이브러리는 여기에서 확인할 수 있습니다.
https://www.kaggle.com/models/google/mobilenet-v2/tfJs 
```

- model을 다운로드받았으면 model을 채울 연료(?)인 bin이 필요하다.
- models.json은 크게 modelTopology와 weightsManifest 옵션으로 나뉜다.
- 그 중 model에 쓰일 bin을 채워넣는 option은 weightsManifest의 paths다.

```
"weightsManifest": [
  {
    "weights": [
        {
          "shape": [],
          "dtype": "float32",
          "name": "ConstantFolding/Postprocessor/Decode/div_recip"
        },
        .
        .
    ],
    "paths": [
          "group1-shard1of5.bin",
          "group1-shard2of5.bin",
          "group1-shard3of5.bin",
          "group1-shard4of5.bin",
          "group1-shard5of5.bin"
    ]
  }
]
 
```

- 그리고 돌려보면 잘 돌아가는 것을 확인할 수 있다.
- 그런데 신한서버에서는 bin파일 확장자를 읽어오지 못했다. 404오류가 떴다.
  - 그래서 txt파일로 변환해봤는데, svn commit을 하려니 오류가 났다.
  - 방법을 찾아보니, 확장자를 바꾸는 게 아니라, 파일을 전부 삭제하고 다시 새로운 확장자로 커밋하라고 한다.
  - 그래서 해당 방식으로 했더니 svn commit 오류는 해결했다.
- 그런데 txt로 바꿔도 여전히 읽어오지 못했다. 404오류가 떴다.
  - 혹시 https의 문제인가 싶어서 살펴보려 했지만, 볼 수가 없었다. http로 해도 https로 redirect해버렸다.
  - 그래서 경로를 절대경로로 바꿔보았다. 

```js
 "paths": [
  "/group1-shard1of5.bin",
  "/group1-shard2of5.bin",
  "/group1-shard3of5.bin",
  "/group1-shard4of5.bin",
  "/group1-shard5of5.bin"
]
```

- 그랬더니 /logic/js/cmm/pet/json//1groupof5.txt로 /가 아니라 //로 변해버렸다.
- 그래서 다시 복원하고 살펴보니, 개발서버의 경로도 /logic/js/cmm/pet/json/1groupof5.txt고, local도 그랬다.

```js
 "paths": [
  "group1-shard1of5.bin",
  "group1-shard2of5.bin",
  "group1-shard3of5.bin",
  "group1-shard4of5.bin",
  "group1-shard5of5.bin"
]
```

- 경로는 전혀 문제가 아니었다. 그래서 txt대신 json으로 만들어서 올렸더니 개발서버에서도 잘 작동했다.
- 파일의 내용이 binary인 경우에는, json 구조를 무시하고 그냥 확장자만 바꿔도 전혀 상관이 없었다.
- 아래같이 json 구조가 아니라 그냥 진짜 쌩 binary 파일이었는데, path를 읽어왔다. 참 다행이었던 순간이다.

```js
[""];
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


## <span style="color:#802548">_tab 이동 시 focus된 버튼 하위 요소를 enter했을 때 문제 해결_</span> 

- focus된 버튼의 경우, enter를 누르면 keydown event에서 enter를 감지한다.
- 그런데 버튼 안에 span 태그에다가 enter를 하는 경우에도 button에서 click event가 일어난다.

```html
<button id="accordion02" class="accordion-btn is-active" type="button" aria-expanded="false"
  aria-controls="layerPanel">
  <span class="accordion-title">
      레이어
      <span class="tooltip-icon">
          <span class="accordion-tooltip">
              한 작업에 관리 가능한 <br>
              레이어 개수는 10개 입니다.
          </span>
      </span>
  </span>
  <!-- <span class="accordion-tooltip">
          한 작업에 관리 가능한 <br>
          레이어 개수는 10개 입니다.
      </span> -->
  <span class="icon icon-arrow"><span class="sr-only">닫힘</span></span>
</button>
```

- 그래서 아래와 같이 span 태그에서 event가 bubbling되지 못하게 막았다.
- 하지만 e.stopPropgation()을 추가해도 여전히 버튼이 클릭되었다.

```js
const accTooltip = document.querySelector('.tooltip-icon');
const accTooltipCon = document.querySelector('.accordion-tooltip');
accTooltip.addEventListener('click', (e) => {
  e.stopPropagation()
  if (!accTooltipCon.classList.contains('is-show')) {
      accTooltipCon.classList.add('is-show')
      accTooltipCon.style.display = 'block'
  } else {
      accTooltipCon.classList.remove('is-show')
      accTooltipCon.style.display = 'none'
  }
})
```

- 그래서 보니 btn에 click listener가 있었다.
- event bubbling 되지 않는 위치일탠데 bubbling이 되는 게 좀 이상하다 싶었는데, click listener가 발동된 것이었다.

```js
btn.addEventListener("click", function () {
  if (acc.classList.contains("onlyone")) {
      // onlyone
      if (this.ariaExpanded === "false") {
          for (let k = 0; k < accBtns.length; k++) {
              accClose(accBtns[k]);
          }
          accOpen(this);
      } else {
          accClose(this);
      }
  } else {
      this.ariaExpanded === "false"
          ? accOpen(this)
          : accClose(this);
  }
});
``` 

- 따라서 span태그가 있는 tooltip에서 event를 막아버렸다.
- event를 if문 밖에서 막으면 되어야할 다른 event가 안되니까, enter일 때만 막도록 하자.
- enter일 떄 event를 막아놓으면 위의 button click event만 막힌다. enter event는 인지가 된다.

```js
const accTooltip = document.querySelector('.tooltip-icon');
const accTooltipCon = document.querySelector('.accordion-tooltip');
accTooltip.addEventListener('click', (e) => {

  if(e.key ==="Enter") {
    e.preventDefault();
     if (!accTooltipCon.classList.contains('is-show')) {
      accTooltipCon.classList.add('is-show')
      accTooltipCon.style.display = 'block'
    } else {
        accTooltipCon.classList.remove('is-show')
        accTooltipCon.style.display = 'none'
    }
  }
})
```


## <span style="color:#802548">_json 파일 읽어오기_</span>

- 욕설을 원래는 서버에서 읽어서 처리했지만, front에서 읽어 처리하는 것으로 결정했다.
- 따라서 json을 front에 두고 처리하기로 했다.
- 코드를 아래와 같이 사용한다.
- isProfanity라는 boolean 변수를 resolve하므로, await로 받게된다면 boolean 변수가 된다.

```js
export const filterProfanity = (text) => {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    
    xhr.open('GET', '/src/assets/profanity.json', true);

    xhr.onload = () => {
      if (xhr.status >= 200 && xhr.status < 300) {
        let profanitys = JSON.parse(xhr.responseText);
        for (const element of profanitys) {
          const isProfanity = text.includes(element);
          if (isProfanity) {
            resolve(isProfanity);
          }
        }
      } else {
        reject();
      }
    }

    xhr.send();
  })
}
```

- json 파일은 아래와 같다.
- 외국어도 들어있어서 ,utf-8로 설정해서 저장해야 된다.

```json
["씨발","개새끼","썅년아","씨발새끼야"....];
```

```js
async function checkProfanity(text) {
  const isProfanity = await filterProfanity(text);

  return isProfanity;
}

const debouncedCheckProfanity = debounce(async (textBox) => {
  const result = await checkProfanity(textBox.text);
  if (result) {
    openModal("욕설꺼져");
    text.set('text,''');
    this.canvas.discardActiveObject();
    this.canvas.setActiveObject(textBox);
    this.canvas.renderAll();
  }
}, 400)
```


## <span style="color:#802548">_fabric.js 비동기 문제_</span>

- fabric.js는 비동기로 실행되는 것들 투성이다.
- 그런데 legacy 환경에서 만들어서 Promise로 감싸져 있지 않아, async await를 쓸 수 없다.
- 비동기로 진행되어 특정 상황에서는 원하는 바가 실현되지 않기도 한다.
  - 예를 들어, 현재 코드는 기존 canvas의 이미지 데이터를 가져와서 씌우고 거기다가 카드를 씌우는 의도다.
  - 그런데, 이미지를 읽는 데 시간이 오래걸리는 경우, 2번째인 cardImage가 먼저 발동되어 2번째 fromURL이 먼저 실행된다.
  - 그리고 나서 1번째 fromURL이 실행되기 때문에, modelCanvas가 clear되고, cardImage를 씌운 부분이 사라지고 동물 이미지(croppedImageData)만 남는다.

```js
fabric.Image.fromURL(croppedImageData, (img) => {
  img.set({
    left: 0,
    top: 0,
    scaleX: modalWidth / viewWidth,
    scaleY: modalHeight / viewHeigth,
    selectable: false,
  })
  modalCanvas.clear();

  modelCanvas.add(img);
})

fabric.Image.fromURL(cardImageUrl, (img) => {
  img.set({
    left: 0,
    top: 0,
    scaleX: modalWidth / viewWidth,
    scaleY: modalHeight / viewHeight,
    selectable: false,
  });

modelCanvas.add(img);
})
```

- 그러한 상황을 방지하기 위해, 무조건 첫번째 fromURL의 add가 먼저 발동되게 해주어야 한다.
- 그를 위해서는 두번째 fromURL, 즉 카드 이미지를 씌우는 과정에 setTimeout을 주어 event loop에 박는다.
- 그럼 첫번째 fromURL의 콜백이 다 발동된 뒤에 두번째 fromURL의 modelCanvas.add(img)가 발동되기 때문에 정상적으로 카드 이미지도 출력된다.
- 원래는 async await를 쓰는게 좋지만, promise를 return하지 않기에 쓸 수가 없어 setTimeout을 써야 한다.

```js
fabric.Image.fromURL(croppedImageData, (img) => {
  img.set({
    left: 0,
    top: 0,
    scaleX: modalWidth / viewWidth,
    scaleY: modalHeight / viewHeigth,
    selectable: false,
  })
  modalCanvas.clear();

  modelCanvas.add(img);
})

fabric.Image.fromURL(cardImageUrl, (img) => {
  img.set({
    left: 0,
    top: 0,
    scaleX: modalWidth / viewWidth,
    scaleY: modalHeight / viewHeight,
    selectable: false,
  });

  setTimeout(()=> {
    modelCanvas.add(img);
  },10)

})
```

## <span style="color:#802548">_회전에 따른 left, top 미 갱신 문제_</span>

- 회전을 하게 되면, left, top 값이 당연히 회전된 객체에 맞춰 정해지는 거라고 생각했다.
- 그런데, 전혀 그렇지 않았다. 원래의 left, top값으로 찍힌 좌표가 회전과 같이 이동했다.
  - left, top이 왼쪽 90도 회전 시에는 회전된 객체에 대해서 left, bottom값을 갖는다.
  - 왼쪽 180도 회전 시에는 회전된 객체에 대해서 right, bottom 값을 갖는다.
  - 왼쪽 270도 회전 시에는 회전된 객체에 대해서 right, top 값을 갖는다.
  - 왼쪽 360도 회전 시에는 다시 돌아온다.
- 이런 상황이다 보니, 회전 시에 그려진 객체에 대한 left, top 값이 측정이 안 됐다.
- 그런 이유로 회전된 이미지 안에서만 cropzone을 만들어서 이동시키는 것이 고장났다.
- 여백을 체크하는 로직도 그러다보니 전부 고장났다.
- 이를 해결하기 위해 여러 시도를 해봤는데, 좌표 값을 갱신하는 코드는 먹지 않았다.
- 좌표값 갱신은 그려진 객체에 대해 left, top점을 유지하면서 좌표 값을 가져왔다.

```js
setCoords();
```

- 결국 방법은 하나였다. left, top 값을 일일이 전부 각도에 맞춰 바꿔주는 것이었다.
- 아래와 같이 일일이 바꿔주었는데, 문제가 생겼다.
- 객체의 속성을 직접 바꿔버리니까 imageObj의 left, top값이 실제로 변동해 render되는 것이었다.
- 캔버스 바깥으로 벗어나 버리는 등 문제가 생겼다.

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


- 따라서 속성 값을 바꾸지 않고, 대리 변수를 사용해 left, top값을 측정하는 로직으로 바꿨다.

```js
const imageObj = this.canvas.getObjects().find((obj) => obj.name?.includes('image'))

let imgLeft = imageObj.left
let imgTop = imageObj.top
if(imageObj.angle === 0 || imageObj.angle === 360 || imageObj.angle === -360 ) {
  imgLeft = imageObj.oCoords.tl.x;
  imgTop = imageObj.oCoords.tl.y;
} else if(imageObj.angle === -90) {
  imgLeft = imageObj.oCoords.tr.x;
  imgTop = imageObj.oCoords.tr.y;
} else if(imageObj.angle === -180) {
  imgLeft = imageObj.oCoords.br.x;
  imgTop = imageObj.oCoords.br.y;
} else if(imageObj.angle === -270) {
  imgLeft = imageObj.oCoords.bl.x;
  imgTop = imageObj.oCoords.bl.y;
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
- 그래서 아래와 같이 

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




## <span style="color:#802548">_카드 여백 체크 로직_</span>

- 원래는 viewWidth로 정한 값을 카드의 left, right, bottom, top 값으로 했다.
- 그런데 실제 값과 달랐다.
- 그래서 아래 같이 background의 left, right, bottom, top을 구하기로 했다.
- cardObj는 회전하지 않기 때문에 분기처리할 필요가 없었다.


```js
const cardObj = this.canvas.getObjects().find((obj) => obj.name?.includes('background'));

const cardLeft = cardObj.oCoords.t1.x;
const cardTop = cardObj.oCoords.t1.y;
const cardRight = cardObj.getScaledWidth() + cardLeft
const cardBottom =  cardObj.getScaledWidth() + cardTop;
```

- 그럼 아래와 같은 분기를 통해 어떤 회전을 하든 background 카드에 여백이 있는지 체크가 가능하다.

```js
if(
  imgTop <= cardTop &&
  imgLeft <= cardLeft && 
  imgRight >= cardRight &&
  imgBottom >= cardBottom
) {

}
```


## <span style="color:#802548">_canvas api의 event는 document의 event와 다르다._</span>

- canvas object에 name 속성을 주는 것은 매우 간단하다.
- fabric.js를 이용한 예시인데, 아래처럼 name을 property로 주면 된다.


```js
const text = new fabric.Textbox('', fontStyleObj);
text.name = _.uniqueId("text_");
this.canvas.centerObject(text);
this.canvas.setActiveObject(text); // 텍스트 상자를 활성 상태로 설정
this.canvas.add(text);
```

- 참고로 fabric.js의 canvas event는 다른 event target과는 다르게 구현되어있다.
- 원래라면 target의 name속성은 HTMLInputElement class의 경우에만 있다. 
- 하지만 여기서 event의 target은 언제나 canvas api 상의 object이므로 name 속성을 부여하기만 하면 가져올 수 있다.

```js
this.canvas.on("object:added", (e) => {
  // 레이어 개수에서 cropZone 제외
  const objects = this.canvas.getObjects();
  // name이 "cropZone"인 객체를 제외한 객체들의 개수
  const count = objects.filter(obj => obj.name !== "cropZone").length;
  .
  .

  const name = e.target.name;
  if (LAYER_IGONORE.includes(name)) return;

});
```

- click event도 아래처럼 준다.

```js
this.canvas.on('mouse:down', function() {

});
```

- canvas만의 특수 event들도 명시할 수 있다.

```js
this.canvas.on('text:changed',function() {

})
this.canvas.on('text:editing:entered',function() {
  
})
```

## <span style="color:#802548">_base64Url을 file객체로 변환하기_</span>

- 서버에서 base64String imageFile을 file객체로 변환해서 넘겨달라고 요청받았다.
- 솔직히 서버에서 알아서 변환하면 되지 않나라는 생각이 들었지만, 네트워크 용량이 큰 건 좋지 않으니 변환을 했다.
- 내가 변환해야 할 base64 image는 fabric.js의 toDataUrl을 사용해서 얻은 값이었다.

```js
const croppedImageData = tempCanvas.toDataURL({
  format: 'jpeg',  // JPEG 형식으로 설정
  left:
      ((centerX - viewWidth / 2) *
          tempCanvas.getZoom() +
          tempCanvas.viewportTransform[4]) + 1,
  top:
      ((centerY - viewHeight / 2) *
          tempCanvas.getZoom() +
          tempCanvas.viewportTransform[5]) + 1,
  width: (viewWidth * tempCanvas.getZoom()),
  height: (viewHeight * tempCanvas.getZoom()),
  multiplier: 7 / tempCanvas.getZoom(),
});
```                        

- 다만 일반적인 base64String image 변환과는 달랐다.
- 원래 일반적으로는 아래와 같은 방식의 image 변환을 거친다.
- 그런데 fabric.js의 base64Url은 자체적으로 안에서 다른 방식을 거쳐서 나오는 듯했다.

```js
let base64 = base64url.replace(/-/g, '+').replace(/_/g, '/');

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

- chatGpt 문의 결과 아래와 같은 방식으로 먼저 앞의 꼭다리를 제거해줘야 한다.
- 그리고 나서는 바이트 배열로 변환하고, blob으로 바꾸고 파일 객체를 만든다.

```js
// Base64 문자열에서 'data:image/png;base64,' 부분 제거
const base64String = dataURL.replace(/^data:image\/(png|jpeg);base64,/, '');

// Base64 문자열을 디코딩하여 바이너리 데이터로 변환하는 함수
function base64ToBinary(base64) {
    const binaryString = atob(base64);
    const len = binaryString.length;
    const bytes = new Uint8Array(len);
    for (let i = 0; i < len; i++) {
        bytes[i] = binaryString.charCodeAt(i);
    }
    return bytes;
}

// 바이너리 데이터 얻기
const binaryData = base64ToBinary(base64String);

// 바이너리 데이터를 사용하여 Blob 생성
const blob = new Blob([binaryData], { type: 'image/png' });

// Blob을 사용하여 File 객체 생성
const file = new File([blob], 'image.png', { type: 'image/png' });
uploadFile(file);
```


- 파일 객체를 만들었다면, 이를 formData에 넣어서 보낸다.

```js
// 파일 업로드 함수
function uploadFile(file) {
  const xhr = new XMLHttpRequest();
  xhr.open("POST", "/upload", true);

  const formData = new FormData();
  formData.append("petPhfile", file); // 원하는 name으로 파일 추가

  xhr.onload = function () {
      if (xhr.status === 200) {
          console.log("Upload successful:", xhr.responseText);
      } else {
          console.error("Upload failed:", xhr.statusText);
      }
  };
  xhr.send(formData);
}
```

- 하지만 우습게도 신한 애들의 서버에서는 null로 자꾸 인식됐다.
- 이유를 몰라. 일단 LmultiPartRequest인지 확인해본다.. 시발
- 일단 확인한 바, form에 넣어 보내야만 인식되는 것으로 확인됐다. 개병신 LG CNS devon을 쓰는 신한카드..

- form에 넣어서 하는 방식을 찾아봤는데, 못찾았다. 
- 근데 우연히도 하나 있었다. 바로 DataTransfer 객체를 사용하는 방식이다.
- DataTransfer를 사용하면, dataTransfer.items.add()를 사용해 마치 drag and drop인 것처럼 속일 수 있다.
- 사용자의 상호작용으로 취급되어 브라우저의 규제를 벗어난다. 나는 사용자가 무조건 선택하는 화면이 나와야 하는 줄 알았다.


```js
const saveTest = (imgUrl) => {
  return new Promise((resolve, reject) => {
    const uploadForm = document.createElement('form');
    uploadForm.id = "frmMultiUpload";
    uploadForm.enctype = "multipart/form-data";

    const fileInput = document.createElement('input');
    fileInput.type = 'file';
    fileInput.id = 'petPhFile';
    fileInput.name = 'petPhFile';
    fileInput.accept = 'image/*';
    fileInput.style.display = 'none';

    const dataTransfer = new DataTransfer();
    dataTransfer.items.add(imgUrl);
    fileInput.files = dataTransfer.files;

    uploadForm.appendChild(fileInput);
    document.body.appendChild(uploadForm);

    const param = new FormData(uploadForm);

    $.ajax({
      type:'post',
      enctype:'multipart/form-data',
      url: '/qq/tttq/rqwr.ajax',
      contentType:'application/x-www-form/urlencoded;charset=utf-8',
      data: param,
      processData: false,
      contentType:false,
      cache: false,
      timeout: 60000,
    })
  })
}
```

## <span style="color:#802548">_multipart/form-data data를 보내기_</span>

- 제이쿼리 ajax에서 contentType을 false로 하면 jquery가 알아서 판단해 보낸다.
- 아래처럼 contentType이 두개면 나중에 쓴 게 적용된다.
- false인데 param이 formData니까 multipart/form-data가 된다.
- 객체를 보낼 때, 문자열로 보내지 않기 위해 processData를 false로 준다. 객체 자체로 보낸다.

```js
$.ajax({
  type:'post',
  enctype:'multipart/form-data',
  url: '/qq/tttq/rqwr.ajax',
  contentType:'application/x-www-form/urlencoded;charset=utf-8',
  data: param,
  processData: false,
  contentType:false,
  cache: false,
  timeout: 60000,
})
```


- xhr로 바꾸면 아래와 같다.
- enctype='multipart/form-data'의 경우는 formData를 쓰면 자동 설정된다.
- formData에 파일을 안 넣고 문자만 넣어도 알아서 encType이 multipart/form-data로 설정된다.
- formData는 사실 contentType을 설정하지 않는게 좋다. 가장 적절한 형태를 알아서 넣는다. 보통 multipart/formData다.
- processData false 옵션도 줄 필요 없다. formData만 넣으면 자동으로 객체 자체로 보낸다.

```js
function sendRequest(param) {
    const xhr = new XMLHttpRequest();
    xhr.open('POST', '/qq/tttq/rqwr.ajax', true);
    xhr.timeout = 60000; // 타임아웃 설정
    xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded;charset=utf-8');

    // onreadystatechange 이벤트 리스너를 설정하여 응답을 처리합니다.
    xhr.onreadystatechange = function() {
        if (xhr.readyState === XMLHttpRequest.DONE) {
            if (xhr.status === 200) {
                console.log('Success:', xhr.responseText);
            } else {
                console.log('Error:', xhr.status, xhr.statusText);
            }
        }
    };

    // ontimeout 이벤트 리스너를 설정합니다.
    xhr.ontimeout = function() {
        console.error('The request for /qq/tttq/rqwr.ajax timed out.');
    };

    // 요청을 보냅니다.
    xhr.send(param);
}

// 예제 호출
const param = new FormData();
param.append('key1', 'value1');
param.append('key2', 'value2');
param.append('file', file);
sendRequest(param);
```


## <span style="color:#802548">_crop시 image 해상도가 깨졌던 이유_</span>

- image 해상도가 깨졌던 이유는 left 값을 round 처리를 안해준 것에 더해,
- border size 값을 생각하지 않은 것 때문이었다. 아래 1은 border size다.

```js
const scaleX = activeSelection.scaleX;
const scaleY = activeSelection.scaleY;

const fixWidth = rect.width - 1; // border size 때문에 값을 빼줘야 정상적으로 보임
const fixHeight = rect.height - 1; // border size 때문에 값을 빼줘야 정상적으로 보임
cropped.src = activeSelection.origin.src;

fabric.Image.fromURL(activeSelection.origin.src, (myImg)=> {
//i create an extra var for to change some image properties
  const url = myImg.toDataURL({
    left: Math.round((rect.left - activeSelection.left) / scaleX),
    top: Math.round((rect.top - activeSelection.top) / scaleY),
    width: Math.round(fixWidth / scaleX),
    height: Math.round(fixHeight / scaleY),
  });
});
```


## <span style="color:#802548">_fabric에서 js로 발생시킨 event는 인식되지 않는다._</span> 

- js로 돌리는 순간, 그냥으로는 event가 발생하지 않는다.
- 아래같이 암만 달아봐야 인지하지 못한다.
- 마우스 클릭을 통한 event로만 인지 가능하다.

```js
this.canvas.on('object:rotated', function(options) {
    options.target.setCoords();
    console.log(options.target);

    let left = null;
    let top = null;
    if(options.target.angle === 0 || options.target.angle === 360 || options.target.angle === -360 ) {
      left = options.target.oCoords.tl.x;
      top = options.target.oCoords.tl.y;
    } else if(options.target.angle === -90) {
      left = options.target.oCoords.tr.x;
      top = options.target.oCoords.tr.y;
    } else if(options.target.angle === -180) {
      left = options.target.oCoords.br.x;
      top = options.target.oCoords.br.y;
    } else if(options.target.angle === -270) {
      left = options.target.oCoords.bl.x;
      top = options.target.oCoords.bl.y;
    }
    console.log(left, top);
  });
```

  - 인지시키기 위해선 강제로 event를 발동시켜줘야 한다.
  - 그럼 위에 걸어둔 rotated event가 인지된다.

  ```js
  this.canvas.fire("object:rotated", {target: activeSelection});
  ```

## <span style="color:#802548">_rotate 돌린 상태로 저장하기_</span>

- 차장게이가 왼90도 돌린 상태로 서버에 전송해달라는 말을 했다.


- 아래처럼 canvas하나를 그냥 새롭게 만들어버리는 게 낫다.
- 원래 있던 canvas를 기반으로 그리려고 해봐야 소용없다..
- 아래처럼 MathPI / 2로 나누면 왼 90도 돌아간 dataurl을 뱉는다.
- -Math.PI / 2로 나누면 오른 90도 돌아간 dataUrl을 뱉는다.
- ToDataUrl()이 비동기라서 Promise 안에 넣어서 async await를 쓸 수 있게 만든다.

```js
function rotateImage90Degrees(imageDataUrl) {
  return new Promise((resolve, reject) => {
    const image = new Image();
    image.onload = () => {
      const width = image.width;
      const height = image.height;
      const canvas2 = document.createElement('canvas');

      if(!(canvas2 instanceof HTMLCanvasElement)) {
        return reject(new Error('failed to create canvas'));
      }

      const ctx = canvas2.getContext('2d');

      canvas2.width = height;
      canvas2.height = width;

      ctx.translate(height / 2, width / 2);
      ctx.rotate(Math.PI / 2);
      ctx.drawImage(image, -width / 2, -height / 2);

      resolve(canvas2.toDataURL());
    }
    image.onerror = reject;
    image.src = imageDataUrl;
  })
}

async function getRotatedImageUrl(imageDataUrl) {
  try {
    const rotateImageUrl = await rotateImage90Degrees(imageDataUrl);
    
    return rotateImageUrl;
  } catch {
    throw error;
  }
}
```


- 그럼 아래와 같이 local에 저장할 수 있다.
- 파일 객체로 만들어 저장한다. 

```js
let saveImg = document.createElement('a');
getRotatedImageUrl(croppedImageData).then((rotatedUrl) => {
  const binaryData = base64UrlToBinary(rotatedUrl);
  const blob = new Blob([binaryData], {type: 'image/jpeg'});
  const file = new File([blob], 'image/jpeg', {type:'image/jpeg'});
  const url = URL.createObjectURL(file);

  saveImg.setAttribute('href', url);
  saveImg.setAttribute('download', "카드.jpg");
  saveImg.click();
  saveImg.remove();
  URL.revokeObjectURL(url);
  tempCavnas.clear();
})
```

- 서버에 보낼 때는 주의할 점이 있다.
- 신한은 legacy 좆병신이라 form-urlencoded 고정으로 보냈다.
- 따라서 body에 넣어 보내도, 전부 string처리되기 때문에, 객체를 그냥 넣으면 절대 안된다.

```js
const data = {
  img: 'img'
};

const xhr = new XMLHttpRequest();
xhr.open('POST', 'your-url-here', true);
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
xhr.send('url=' + data);
```

- 위와 같이 url이란 name에 data를 객체로 보내면 제대로 인식되지 못한다.

```
url=[object object]
```

- formurlencoded로 보내려면 아래처럼 encodeURIComponent로 감싸서 보내줘야 한다.

```js
const data = {
  img: 'img'
};

// URL-encoded 형식으로 변환
const encodedData = 'json=' + encodeURIComponent(data);

// XMLHttpRequest 객체 생성
const xhr = new XMLHttpRequest();
xhr.open('POST', 'your-url-here', true);
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');

// 데이터 전송
xhr.send(encodedData);
```


- json문자열로 바꿀거라면 아래처럼 stringify도 넣는다.

```js
const data = {
  img: 'img'
};

// JSON 문자열로 변환
const jsonData = JSON.stringify(data);

// URL-encoded 형식으로 변환
const encodedData = 'json=' + encodeURIComponent(jsonData);

// XMLHttpRequest 객체 생성
const xhr = new XMLHttpRequest();
xhr.open('POST', 'your-url-here', true);
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');

// 데이터 전송
xhr.send(encodedData);
```



## <span style="color:#802548">_tomcat의 post 전송 시 body 크기 2MB 제한_</span>

- 우선 tomcat은 post로 전송시에, 2mb가 넘으면 그냥 튕겨버린다. 그게 default다.
- 따라서 controller에는 값이 들어오지 않는다. controller 자체에는 도달한다.
- server.xml에 8080 connector에 maxPostSize="104786760" 같이 10MB로 올려줘야 2MB가 넘는 것도 들어온다.
- base64UrlString은 2MB가 넘는 경우가 꽤 많으니 해당 부분을 잘 확인해야 한다.
- 그런데 문제는 2MB를 넘기게 설정해도, 읽어오는데 하세월이라는 점이다.
- 2MB넘는 base64String을 읽어오는데 3분 정도가 걸렸다. 이건 써먹을 수가 없다.




## <span style="color:#802548">_js,css,font caching 방지하기_</span>

- js script를 갈아끼워도 caching된 것이 남아있으면 caching된 것을 읽는다.
- 따라서 변화를 로컬 혹은 운영에 반영해도 읽어오지 못하는 경우가 있다.
- 이는 legacy도 마찬가지지만, npm 기반의 vue나 react도 마찬가지다.


- vue의 경우, vue.config.js에 아래 내용을 추가해준다.

```js
module.exports = {
  chainWebpack: (config) => {
    config.output.filename('js/[name].[hash].js').end();
    config.output.chunkFilename('js/[name].[hash].js').end();
  },
};
```

- vite기반의 경우, 기본적으로 파일에 hash가 추가되므로 알아서 방지된다.



- 개발 시 특별한 옵션을 넣기 싫다면, 로컬에서는 그냥 개발자도구창을 열고 새로고침에 오른쪽 마우스 클릭하고 캐시 비우고 새로고침을 누른다.
- js 방식으로 처리한다면 아래와 같다. html에서 script를 불러올 때 아래와 같이 날짜를 추가해준다.
- src끝에 querystring을 달면 처음 반영시에는 caching이 방지될 지 몰라도, 똑같이 유지하면서 새로운 내용을 배포할 때는 caching을 방지하지 못한다.

```html
<script src="fqawrqwrrq?2015241"></script>
```


- 따라서 동적으로 아래와 같이 불러주는 js를 따로 만들수도 있다.
- 안바뀐다는 확신이 있으면 version을 안 추가해줘도 된다.

```js
//photo.js
(function() {
  const version = new Date().getDate() + '' + (new Date().getHours());
  document.write('<link rel="preload" as="font/woff2" href=/pconts/fonts/shcard/Srrr.woff2" corssorigin />\n'
  + '<link rel="preload" as="style"  href="/pconts/css/shcard/fonts.css?ver=' + ver +'" />\n'
  + '<link rel="preload" as="style"  href="/pconts/css/shcard/fonts.css?ver=20230114" />\n'
  + '<link rel="preload" as="script"  href="/pconts/css/shcard/photoFunction.js?ver=' + ver +'" />\n'


  )
})();
```

- caching방지용으로 만들어둔 js를 html에 넣는다.
- 제일 위에다가 놓아주자.

```html
<head>
  <script src="/function/photo.js"></script>

  <script src="/libary/tf.js"></script>
  <script src="/libary/coco-ssd.js"></script>
  <script src="/libary/fabric.js"></script>
</head>
<body>
  .
  .
</body>
```


## <span style="color:#802548">_filter와 find의 차이점_</span>

- filter를 쓰면 배열로 취급된다.
- 따라서 index를 넣어서 가져와야 한다.
- 만약 0번쨰를 설정하지 않고 진행하면 콘솔창에 오류는 안뜨는데, 오류는 찾을 수 없는 상황이 되어버린다.
- 따라서 find과 filter의 차이를 반드시 외워두자. find는 객체, filter는 array다.

```js
const leftLengthDisplayingText = this.canvas.getObjects().filter((item)=> item.name.includes("text_length"));
leftLengthDisplayingText[0].text = textBox.text.length;
```

- 반면에 find를 쓰면 object로 취급된다.
- 따라서 index를 넣지 않고 가져와야 한다.
- 여기서는 0번쨰를 설정하지 않고 진행해야 한다.

```js
const leftLengthDisplayingText = this.canvas.getObjects().filter((item)=> item.name.includes("text_length"));
leftLengthDisplayingText.text = textBox.text.length;
```

- 또한 text를 설정하는 순간, 자동으로 split이 일어난다.
- 그런데 length 속성은 number type이므로 오류가 난다.
- 따라서 형변환을 해줘야 한다.

```js
leftLengthDisplayingText.text = String(textBox.text.length);
```

## <span style="color:#802548">_typing event가 일어나기 전의 텍스트 저장하기_</span>

- 우선 텍스트에 관한 요건이 매우 복잡했다. 
- 이전에 입력한 text 언어와 현재 입력한 text 언어의 양식이 다르면 막아달라는 것이었다.
- 다만 이전text와 현재 입력된 text의 값이 달을 구분해서 진행해야 했으므로 flag값을 추가했다.
- 여기서 새로 추가된 text를 가져오려면, 이전 text만큼을 빼줘야 했다. 따라서 previousTextLength - 1이 된것이다.
- 그러면서 원본 객체에는 영향이 가지 않아야 했으므로 slice가 아닌 substring을 사용해 새로운 문자열을 만들었다.
- 현재입력된 text는 과거 text의 길이만큼 잘라야 하므로 잘라서 가져온다.
- 다만 \uAC00-\uD7AF정규식을 사용하니 한글 ㄱ이라던가 자음 모음 하나씩만 입력했을 때 한국어 체크가 안돼서 아래와 같이 변경했다.
- 다만 전부지워서 0이 되는 경우에는 한글, 영어 어디로든 갈 수 있으니 정규식 검사를 하지 않게 했다.

```js
//한글이면 true, 아니면 false
function verifyKoreanOrNot(text, flag) {
  const previousTextLength = text.length;
  const currentEventText = text.substring(previousTextLength - 1); //slice쓰면 원본객체도 영향을 받기때문에 쓰지 않음. 

  if(text.length !== 0 && flag === 'prev') {
    return /^[ㄱ-ㅎ가-힣ㅏ-ㅣ]+$/.test(text);
  } else if(currentEventText.length !== 0 && flag ==='cur') {
    return /^[ㄱ-ㅎ가-힣ㅏ-ㅣ]+$/.test(currentEventText);
  }
}

//영어면 true, 아니면 false
function verifyEnglishOrNot(text, flag) {
  const previousTextLength = text.length;
  const currentEventText = text.substring(previousTextLength - 1); //slice쓰면 원본객체도 영향을 받기때문에 쓰지 않음. 0번째 index부터니 1을 빼줌.

  if(text.length !== 0 && flag === 'prev') {
    return /^[a-zA-Z]+$/.test(text);
  } else if(currentEventText.length !== 0 && flag ==='cur') {
    return /^[a-zA-Z]+$/.test(currentEventText);
  }
}
```


- 이제 해당 정규식을 이용해 과거의 text와 현재 입력된 text의 언어를 검사하게했다.
- 한 -> 영, 영 -> 한이면서 이전 text의 값이 있는 경우에만 언어 전환을 막는 로직을 발동시킨다.
- 다만 return;을 쓰지 못하고 복잡하게 boolean 변수로 process를 control했다.
  - 그 이유는 어쨌든 이전의 요건을 통과한 경우에는 prevText값은 무조건 보관해야 하기 때문이다.
- 한->영, 영->한 전환 시 해당 문자는 입력하지 않은 처리를 하므로, 문자열 길이 보여주는 text도 입력하지 않은 처리를 반영했다.
- 맨 마지막에 prevText 값을 저장하게 하여 로직을 사용할 때는 prevText값을 사용할 수 있게 적용하였다.
  - 다만 text가 이전 text와 같은 경우에는 굳이 판별할 필요가 없으므로 제외한다.

```js

const self = this;
function refrainKorEngMixing(previousText, textBox) {
  let isPrevTextKorean = verifyKoreanOrNot(previousText.text, 'prev');
  let isCurrentEventTextEnlgish = verifyEnglishOrNot(textBox.text, 'cur');
  let isPrevTextEnglish = verifyEnglishOrNot(previousText.text, 'prev');
  let isCurrentEventTextKorean = verifyKoreanOrNot(textBox.text, 'cur');


  let isKorToEng = (isPrevTextKorean  && isCurrentEventTextEng);
  let isEngToKor = (isPrevTextEnglish && isCurrentEventTextKorean)
  if (isKoreToEng || isEngToKor) {
    let isContinuingLogic = true;

    if (!previousText.text) {
      isContinuingLogic = false;
    }

    if (isContinuingLogic) {
      textBox.text = textBox.text.slice(0, textBox.text.length - 1);

      //한->영, 영->한 전환 시 해당 문자는 입력하지 않은 처리를 하므로, 문자열 길이 보여주는 text도 입력하지 않은 처리를 반영
      if(textBox.name?.includes('left')) {
        const leftLengthDisplayText = self.canvas.getObjects().find((item) => item.name.includes('text_length_left'));
        leftLengthDisplayText.text = String(textBox.text.length);
      } else {
        const centerLengthDisplayText = self.canvas.getObjects().find((item) => item.name.includes('text_length_center'));
        centerLengthDisplayText.text = String(textBox.text.length);
      } 

      self.canvas.discardActiveObject();
      self.canvas.setActiveObject(textBox);
    }

  }

  //이전텍스트 값을 저장
  if (previousText.text !== textBox.text) {
    previousText.text = textBox.text;
  }
}
```

- 하지만 위의 방식으론 공백의 경우를 잡을 수가 없다.
- 우선 입력 시에도 공백을 허용하게끔 정규식에 공백도 추가해줘야한다.

```js
this.cavnas.on('text:changed', (e) => {

  if (textBox.name?.includes('left')) {
    const regExp = /[^ㄱ-ㅎ가-힣ㅏ-ㅣa-zA-Z\u3180\u119E\u11A2\u2022\u2025a\u00B7\uFE55\s]+$/gi;
    const isValidText = filterTextRetrieveValidationResult(regExp, textBox);

    if (!isValidText) {
      return;
    }

    limitLeftTextAmount(textBox);
    refrainKorEngMixing(previousLeftText, textBox);
  }
})
```

- 공백을 받는 경우, 공백으로 인해 정규식이 전부 깨져버린다. 따라서 kor-eng 용 정규식에 넣을 텍스트는 공백을 전부 제거해준다.
- 또한 현재 들어온 event 값이 공백인 경우, 이전에 보존한 텍스트 값을 바꾸지 않는 조건을 추가한다. 텍스트값에 공백이 포함되면 안되기 때문이다.
- 이벤트 텍스트값도 trim한 값으로 만든다. 막 입력된 event input도 공백이 들어올 수 있기 때문이다.

```js
const self = this;
function refrainKorEngMixing(previousText, textBox) {
  let isPrevTextKorean = verifyKoreanOrNot(previousText.text.trim(), 'prev');
  let isCurrentEventTextEnlgish = verifyEnglishOrNot(textBox.text.trim(), 'cur');
  let isPrevTextEnglish = verifyEnglishOrNot(previousText.text.trim(), 'prev');
  let isCurrentEventTextKorean = verifyKoreanOrNot(textBox.text.trim(), 'cur');
    if (!previousText.text.trim()) {
      isContinuingLogic = false;
    }
.
.
.
  //이전텍스트 값을 저장. 공백값은 저장하지 않고 한글, 영어로만 값을 간직한다.
  if (previousText.text !== textBox.text && !textBox.text.includes(' ')) {
    previousText.text = textBox.text;
  }
}
```



- this를 쓰는 방식은 self = this 말고 아래와 같이도 쓸 수 있다.
- 매개변수가 있어도 아래와 같이 bind로 묶는 게 가능한 이유는 매개변수는 그대로 두고 this만 binding하는 것이라 그렇다.

```js
function refrainKorEngMixing(previousText, textBox) {
    this.canvas.discardActiveObject();
    this.canvas.setActiveObject(textBox);
}

refrainKorEngMixing = refrainKorEngMixing.bind(this);
```

## <span style="color:#802548">_object와 원시형의 차이점_</span>

- slice는 원본을 잘라버린다.
- slice가 원본을 자르는 특성이 필요할 때가 있다.
- 원본의 값을 바꿔야만 할 때다.

- 그런데 그냥 바꾸는 건 적용되지 않고, 객체의 property로 만들어서 변경해줘야 한다.
- 아래처럼 property로 만들어서 바꾸지 않으면, 바뀌지 않는다.
- 원본을 함수의 parameter로 넘기는 순간, function stack에서 자체적으로 값을 만들어 사용하기 때문이다.
- 아래처럼 만들어 버리면 previousText 값이 바깥에서 바뀌지 않는다.

```js
let previousLeftText = leftDefaultText
let previoustCenterText = centerDefaultText

function refrainKorEngMixing(previousText, textBox) {
  .
  .
  .
  if (previousText !== textBox.text) {
    previousText = textBox.text;
  }
}
```


- 아래처럼 객체의 property로 저장해야 바깥에서도 바뀐 값을 사용할 수 있다.

```js
let previousLeftText = {
  text: ''
}
let previoustCenterText = {
  text: ''
}

function refrainKorEngMixing(previousText, textBox) {
  .
  .
  .
  if (previousText.text !== textBox.text) {
    previousText.text = textBox.text;
  }
}
```

## <span style="color:#802548">_replace, slice/ 대리변수, substring_</span>

- 다만 이때 조심해야 할 점이 있는데, 바로 slice와 substring을 사용할 때를 구분하는 것이다.
- slice는 객체 자체 원본 property를 바꿔버린다. 따라서 원본 값이 바뀌는 걸 원하지 않으면 사용해선 안된다.
- 원본 값이 바뀌는 걸 원하지 않는다면, substring으로 가져와서 사용하거나, 대리변수를 만들어야 한다.
- 대리변수를 쓰는 방법은 아래 예시를 참고하자. 만약 대리변수를 쓰지 않으면 실제 이미지의 left 값이 바뀌어버린다.

```js
const imageObj = this.canvas.getObjects().find((obj) => obj.name?.includes('image'))

let imgLeft = imageObj.left
let imgTop = imageObj.top
if (imageObj.angle === 0 || imageObj.angle === 360 || imageObj.angle === -360 ) {
    imageObj.left = imageObj.oCoords.tl.x;
    imageObj.top = imageObj.oCoords.tl.y;
} 
```                  


- 따라서 속성 값을 바꾸지 않고, 대리 변수를 사용해 left, top값을 측정하는 로직으로 바꿨다.

```js
const imageObj = this.canvas.getObjects().find((obj) => obj.name?.includes('image'))

let imgLeft = imageObj.left
let imgTop = imageObj.top
if(imageObj.angle === 0 || imageObj.angle === 360 || imageObj.angle === -360 ) {
  imgLeft = imageObj.oCoords.tl.x;
  imgTop = imageObj.oCoords.tl.y;
} 
```   

- substring을 쓰는 것은 아래와 같다.
- substring을 쓰면, textBox의 property인 text의 값은 건드리지 않기 때문에 문제가 없다.
- 실제 event가 일어난 현재 text는 'textㄱ'고, 실제 입력된 값은 'ㄱ'라고 해보자.
  - 현재 text인 'textㄱ'는 보존하고, 실제 입력된 값인 'ㄱ'만으로 한글-영어 여부 판별이 필요하다.
  - 따라서 substring으로 새로운 문자열을 만드는 것이다.

```js
function verrifyEnglishOrNot(text, flag) {
  const previousTextLength = text.length;
  const currentEventText = text.substring(previousTextLength - 1); //slice쓰면 원본객체도 영향을 받기때문에 쓰지 않음. 0번째 index부터니 1을 빼줌.

  if(text.length !== 0 && flag === 'prev') {
    return /^[a-zA-Z]+$/.test(text);
  } else if(currentEventText.length !== 0 && flag ==='cur') {
    return /^[a-zA-Z]+$/.test(currentEventText);
  }
}
```

- 반면 replace, slice를 써야하는 상황도 있다.
- 정규식을 통해 값을 확인해 걸러내는 경우에 필요하다.
- 객체 원본 property 값 자체를 바꿔줘야 하는 경우 아래처럼 replace를 활용한다.

```js
const regExp = /[^ㄱ-ㅎ가-힣ㅏ-ㅣa-zA-Z\u3180\u119E\u11A2\u2022\u2025a\u00B7\uFE55]+$/gi;
  if (regExpt.test(textBox.text)) {
    textBox.text = textBox.text.replace(regExp, '');
    this.canvas.discardActiveObject();
    this.canvas.ssetActiveObject(textBox);
    return;
  }
```

- slice를 활용하는 경우는 방금 입력한 값만 삭제하려고 할 때 많이 쓴다.
- 글자의 최대 길이 제한이 있는 경우에 maxLength를 알기 쉬우니 특히 많이 쓴다.
- 최대 제한이상의 글자를 쓰면, 글자수가 늘어나는 것을 글자수 text에 반영해줄 필요가 없다.
- 따라서 최대제한 if문에는 적용하지 않고, 그 외의 경우에만 적용시킨다.
- 문자열을 discard하고 다시 active 시키는 이유는 그렇게 안하면 편집모드 상태에선 text가 입력된 값을 그대로 보존하기 때문이다.

```js
const limitLeftTextAmount = (textBox) => {
  if (textBox.text.length > 7) {
    textBox.text = textBox.text.slice(0, 7);
    this.canvas.discardActiveObject();
    this.canvas.setActiveObject(textBox);
  }  else {
    const leftLengthDisplayText = self.canvas.getObjects().find((item) => item.name.includes('text_length_left'));
    leftLengthDisplayText.text = String(textBox.text.length);
  }
}
```


## <span style="color:#802548">_text 검사 순서와 정규식_</span>
- text에 관한 복잡한 로직은 이제 거의 정리됐다.
- 먼저 욕설 검사를 먼저 진행한다.
- 그리고 canvas자체에 event를 거는 것말곤 도리가 없으므로, event의 name을 보고 분기처리한다.
- 분기처리를 하는경우 가장 먼저 한영만 입력가능한지, 숫자만 입력가능한지 확인한다.
- 만약 해당 정규식에 위반되면 그 이후 로직인 최대숫자 제한, 한-영 영-한 전환 금지 로직은 실행조차 하지 않는다.
- 그냥 그전에 먼저 문자열을 삭제시켜버린다.
- 해당 문자 정규식 검사도 함수로 따로 뺄 수 있는데, 대신 빼게 되면 함수안에서 return시키기가 귀찮아진다.

```js
this.cavnas.on('text:changed', (e) => {
  const textBox = e.target;
  debouncedCheckProfanity(textBox);

  if (textBox.name?.includes('left')) {
    const regExp = /[^ㄱ-ㅎ가-힣ㅏ-ㅣa-zA-Z\u3180\u119E\u11A2\u2022\u2025a\u00B7\uFE55]+$/gi;
    if (regExpt.test(textBox.text)) {
      textBox.text = textBox.text.replace(regExp, '');
      this.canvas.discardActiveObject();
      this.canvas.ssetActiveObject(textBox);
      return;
    }
    limitLeftTextAmount(textBox);
    refrainKorEngMixing(previousLeftText, textBox);
  } else if(textBox.name?.includes('center')) {
    const regExp = /[^ㄱ-ㅎ가-힣ㅏ-ㅣa-zA-Z\u3180\u119E\u11A2\u2022\u2025a\u00B7\uFE55]+$/gi;
    if (regExpt.test(textBox.text)) {
      textBox.text = textBox.text.replace(regExp, '');
      this.canvas.discardActiveObject();
      this.canvas.ssetActiveObject(textBox);
      return;
    }
    limitLeftTextAmount(textBox);
    refrainKorEngMixing(previousLeftText, textBox);
  } else {
    const regExp = /[^0-9]+$/gi;
    if (regExpt.test(textBox.text)) {
        textBox.text = textBox.text.replace(regExp, '');
        this.canvas.discardActiveObject();
        this.canvas.ssetActiveObject(textBox);
        return;
      }
      limitRightTextAmount(textBox);
    }
})
```


- 그래도 아래처럼 좀 깔끔해지는 효과가 있다.
- 핵심 식들에 이름을 주니까 이해하기 쉬워진다.

```js
function filterTextRetrieveValidationResult(regex, textBox) {
  if (regex.test(textBox.text)) {
      textBox.text = textBox.text.replace(regex, '');
      this.canvas.discardActiveObject();
      this.canvas.ssetActiveObject(textBox);
      return false;
    }

    return true;
}

this.cavnas.on('text:changed', (e) => {
  const textBox = e.target;
  debouncedCheckProfanity(textBox);

  if (textBox.name?.includes('left')) {
    const regExp = /[^ㄱ-ㅎ가-힣ㅏ-ㅣa-zA-Z\u3180\u119E\u11A2\u2022\u2025a\u00B7\uFE55]+$/gi;
    const isValidText = filterTextRetrieveValidationResult(regExp, textBox);

    if (!isValidText) {
      return;
    }

    limitLeftTextAmount(textBox);
    refrainKorEngMixing(previousLeftText, textBox);
    
  } else if(textBox.name?.includes('center')) {
    const regExp = /[^ㄱ-ㅎ가-힣ㅏ-ㅣa-zA-Z\u3180\u119E\u11A2\u2022\u2025a\u00B7\uFE55]+$/gi;
    const isValidText = filterTextRetrieveValidationResult(regExp, textBox);

    if (!isValidText) {
      return;
    }

    limitLeftTextAmount(textBox);
    refrainKorEngMixing(previousCenterText, textBox);
  } else {
    const regExp = /[^0-9]+$/gi;
    const isValidText = filterTextRetrieveValidationResult(regExp, textBox);

    if (!isValidText) {
      return;
    }

    limitRigthTextAmount(textBox);
  }
})
```


- 그런데 공백을 추가하는 과정에서 계속 오류가 나기 시작했다.
- 그래서 원인을 찾기 위해 regex를 콘솔로 찍어보았다.
- 그런데 말도 안되게 위의 콘솔 regex test는 true인데, if문의 regex test는 false가 나왔다.

```js
function filterTextRetrieveValidationResult(regex, textBox) {
    console.log("regex.test(textBox.text)", regex.test(textBox.text));
  if (regex.test(textBox.text)) {
      textBox.text = textBox.text.replace(regex, '');
      this.canvas.discardActiveObject();
      this.canvas.ssetActiveObject(textBox);
      return false;
    }

    return true;
}
```

- 원인을 알아보니 regex는 g flag가 있는 경우, 경우에 따라 regex의 lastIndex가 달라지기 때문이라고 한다.
- 따라서 regex를 호출할 때는 console.log로 값을 보면 안되는 것이었다.
- 그래서 식을 다시 원복했다.

```js
function filterTextRetrieveValidationResult(regex, textBox) {
  if (regex.test(textBox.text)) {
      textBox.text = textBox.text.replace(regex, '');
      this.canvas.discardActiveObject();
      this.canvas.ssetActiveObject(textBox);
      return false;
    }

    return true;
}
```

- 원인은 정규식 중 $의 문제였다. $는 문자열 끝만을 의미하는 것이다.
- 다시말해 막 입력한 값에 대해서만 정규식이 발동하고 있던 셈이었다.
- 아래 정규식은 문자열의 끝 부분에서 한글이 아닌 모든 문자를 찾고, 이 문자들을 제거하는 형태였다.

```js
const regex = /[^ㄱ-ㅎ가-힣ㅏ-ㅣ]+$gi/;
textBox.text = textBox.text.replace(regex, '');
```

- '서브 텍스트 입력' 중 서브만 더블클릭해 숫자로 바꾸는 경우, 전체 문자에 대해 정규식 적용이 필요했다.
- 따라서 문자열 끝만을 의미하는 $를 지웠다. 그러자 원하는대로 정규식이 작동한다.

```js
const regex = /[^ㄱ-ㅎ가-힣ㅏ-ㅣ]+gi/;
```

- '서브 텍스트 입력' 일 때 앞의 서브만 더블클릭해서 숫자를 넣는 경우에, 지워지면서 1은 입력되지 않게 완성했다.

```js
const regex = /[^ㄱ-ㅎ가-힣ㅏ-ㅣ]+gi/;
function filterTextRetrieveValidationResult(regex, textBox) {
  if (regex.test(textBox.text)) {
      textBox.text = textBox.text.replace(regex, '');
      this.canvas.discardActiveObject();
      this.canvas.ssetActiveObject(textBox);
      return false;
    }

    return true;
}
```





## <span style="color:#802548">_폰트사이즈에 맞춰 height 유지시키기_</span>

- 폰트사이즈가 커질수록, height도 높아져 원래 높이를 유지하지 못하고 글자가 아래로 내려간다.
- 이런 현상을 방지하려면 fontSize에 맞춰 top 크기도 조정되어야 한다.

```js
function limitCenterTextAmount (textBox) {
  const previousFontSize = textBox.fontSize;

  if(textBox.text.length === 1) {
    textBox.fontSize = 52;
  } else if(textBox.text.length === 2) {
    textBox.fontSize = 42;
  } else if(textBox.text.length === 3) {
    textBox.fontSize = 38;
  } else if(textBox.text.length === 4) {
    textBox.fontSize = 33;
  } else if(textBox.text.length === 5) {
    textBox.fontSize = 28;
  } else if(textBox.text.length === 6) {
    textBox.fontSize = 25;
  } else if(textBox.text.length === 7) {
    textBox.fontSize = 22;
  } else if(textBox.text.length === 8) {
    textBox.fontSize = 18;
  }

  textBox.top = textBox.top - (previousFontSize - textBox.fontSize);
}
```

- 위를 ES6 버전으로 바꾸면 아래와 같다.
- property의 속성을 바꾸려면 destructuring이 아니라 직접 dot notation으로 접근해야 한다.
- destructuring은 local variable을 만드는 것 뿐이다.

```js
function limitCenterTextAmount(textBox) {
  const { fontSize: previousFontSize, text } = textBox;
  const { length } = text;
  
  if (length === 1) {
    textBox.fontSize = 52;
  } else if (length === 2) {
    textBox.fontSize = 42;
  } else if (length === 3) {
    textBox.fontSize = 38;
  } else if (length === 4) {
    textBox.fontSize = 33;
  } else if (length === 5) {
    textBox.fontSize = 28;
  } else if (length === 6) {
    textBox.fontSize = 25;
  } else if (length === 7) {
    textBox.fontSize = 22;
  } else if (length === 8) {
    textBox.fontSize = 18;
  }

  textBox.top = textBox.top - (textBox.fontSize - previousFontSize);
}
```

- 아래와 같이 만들면 실제로 local variable은 previousFontSize가 변수명이 된다.

```js
const { fontSize: previousFontSize, text } = textBox;

//
const previousFontSize = textBox.fontSize;
const text = textBox.text;
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


## <span style="color:#802548">_api timeout 시 재시도하는 로직_</span>

- model을 load해서 개/고양이 판독 api를 쏘는데, 이 경우 외부망을 통해 나간다.
- 그런데 구글이 잘못돼서(???) 혹시 개/고양이 판독이 실패한 경우, 내부 모델을 활용해 판독을 하게끔 하는 요건이 있었다.
- 그 경우 timeout 함수를 만든다. 성공이나 실패 시 타이머를 없앤다.

```js
function withTimeout(promise, timeout) {
  return new Promise((resolve, reject) => {
    const timer = setTimeout(() => {
      reject(new Error('Operation timed out'));
    }, timeout)

    promise.then((value)=> {
      clearTimeout(timer);
      resolve(value);
    }).catch((err)=> {
      clearTimeout(timer);
      reject(err);
    })
  })
}
```

- 좀 복잡한 식이지만, 처음 api 전송이 2초안에 성공이든 실패든 응답이 없으면 내부 모델을 활용하는 식이다.

```js
try {
  if (!loadedModel) {
    loadedModel = await withTimeout(cocoSsd.load(), 2000);
  }

  return loadedModel;
} catch (error) {
  try {
    //cocoSsd 내부 모델 로드
    if (!loadedModel) {
      loadModel = await cocoSsd.load({
        base: 'lite_mobilenet_v2',
        modelUrl: '/pet/js/json/models.json'
      })
    }
    return loadedModel;
  } catch (error) {
    console.log('load model internal Error');
  }
} finally {
  const model = loadedModel;
  // const model = await cocoSsd.load();

  // 이미지를 텐서로 변환
  const tensor = tf.browser.fromPixels(imgObj);

  // 모델에 이미지 전달하여 예측
  const predictions = await model.detect(imgObj);

  // 예측 결과 출력
  console.log(predictions);

  // 예측 결과 중 가장 높은 확률의 라벨 가져오기
  const topPrediction = predictions[0]?.class;

  return topPrediction;
}
```

- 더 간단하게 아래처럼 바꿀 수 있다.

```js
async function loadAndPredict(imgObj) {
  try {
    if (!loadedModel) {
      try {
        loadedModel = await withTimeout(cocoSsd.load(), 2000);
      } catch {
        // 기본 모델 로드 시도
        loadedModel = await cocoSsd.load({
          base: 'lite_mobilenet_v2',
          modelUrl: '/pet/js/json/models.json'
        });
      }
    }

    // 이미지를 텐서로 변환
    const tensor = tf.browser.fromPixels(imgObj);

    // 모델에 이미지 전달하여 예측
    const predictions = await loadedModel.detect(imgObj);

    // 예측 결과 출력
    console.log(predictions);

    // 예측 결과 중 가장 높은 확률의 라벨 가져오기
    const topPrediction = predictions[0]?.class;

    return topPrediction;
  } catch (error) {
    console.log('Model load or prediction error:', error);
    return null;
  }
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

## <span style="color:#802548">_텍스트 길이를 보여주는 text도 만들기_</span>

- 텍스트 길이를 보여주는 text도 만들어 달라는 추가요건이 들어왔다.
- 텍스트 길이는 처음엔 파란색이다가 최대글자 수를 초과하면 빨간색으로 노출시켜달라는 요건이었다.
- 다만 디자인 사양을 받은 건 없어 일단 임의로 내가 맞췄다. 나중에 다시 바꿔야될 거 같다.
- select는 불가능해야 한다.

```js
const leftLengthTextLeftDistance = 40;
const leftLengthTextTopDistance = 6;
const leftLengthDisplayingText = new fabric.TextBox('0/7', {
  fontSize: bottomSize,
  fill: 'blue',
  fontFamilty : "Malgun Gothic",
  name: "text_length_left",
  width: 40,
  left: leftTextBox.left,
  top: leftTextBox.top,
  height: 100,
  textAlign: 'left',
  selectable: false,
  ...defaultOptions
})
leftLengthDisplayingText.set('left', leftLengthDisplayingText.left + leftLengthTextLeftDistance );
leftLengthDisplayingText.set('top', leftLengthDisplayingText.top + leftLengthTextTopDistance );
```

- 오른쪽도 똑같이 만들어준다.

```js
const rightLengthTextLeftDistance = 40;
const rightLengthTextTopDistance = 6;
const rightLengthDisplayingText = new fabric.TextBox('0/18', {
  fontSize: bottomSize,
  fill: 'blue',
  fontFamilty : "Malgun Gothic",
  name: "text_length_left",
  width: 40,
  left: leftTextBox.left,
  top: leftTextBox.top,
  height: 100,
  textAlign: 'left',
  selectable: false,
  ...defaultOptions
})
rightLengthDisplayingText.set('left', rightLengthDisplayingText.left + rightLengthTextLeftDistance );
rightLengthDisplayingText.set('top', rightLengthDisplayingText.top + rightLengthTextTopDistance );
```

- center도 똑같이 만들어준다.

```js
const centerLengthTextLeftDistance = 110;
const centerLengthTextTopDistance = 15;
const centerLengthDisplayingText = new fabric.TextBox('0/18', {
  fontSize: bottomSize,
  fill: 'blue',
  fontFamilty : "Malgun Gothic",
  name: "text_length_left",
  width: 40,
  left: leftTextBox.left,
  top: leftTextBox.top,
  height: 100,
  textAlign: 'left',
  selectable: false,
  ...defaultOptions
})
centerLengthDisplayingText.set('left', centerLengthDisplayingText.left + centerLengthTextLeftDistance );
centerLengthDisplayingText.set('top', centerLengthDisplayingText.top + centerLengthTextTopDistance );
```


- 해당 텍스트 길이를 보여주는 text는 레이어에 추가하지 않게끔 한다.
- 레이어에 필요하지 않은 text기 때문이다.

```js
const LAYER_IGNORE = [
  "text_length_left",
  "text_length_right",
  "text_length_center",
]

this.canvas.on("object:added", (e) => {
  // 레이어 개수에서 cropZone 제외
  const objects = this.canvas.getObjects();
  // name이 "cropZone"인 객체를 제외한 객체들의 개수
  const count = objects.filter(obj => obj.name !== "cropZone").length;
  .
  .

  const name = e.target.name;
  if (LAYER_IGONORE.includes(name)) return;

});
```

- 이제 텍스트 길이를 제한하고 최대글자제한수를 넘으면 글자가 빨개지는 로직을 도입한다.
- else문에 들어서는 순간 다시 정상 글자수이기에, 파란글씨로 바꾸고 글자수 변화를 반영한다.

```js
const limitLeftTextAmount = (textBox) => {
  const languageMaxLength = 7;
  const leftLengthDisplayingText = this.canvas.getObjects().find((item) => item.name.includes('text_length_left'));
  if (textBox.text.length > languageMaxLength) {
    leftLengthDisplayingText.set({fill:'red'});
    textBox.text = textBox.text.slice(0, languageMaxLength );
    this.canvas.discardActiveObject();
    this.canvas.setActiveObject(textBox);
  } else {
    leftLengthDisplayingText.set({fill:'blue'});
    leftLengthDisplayingText.text = String(textBox.text.length + '/' + languageMaxLength);
  }
}

const limitCenterTextAmount = (textBox) => {
  const languageMaxLength = 8;
  const centerLengthDisplayingText = this.canvas.getObjects().find((item) => item.name.includes('text_length_center'));
  if (textBox.text.length > languageMaxLength) {
    centerLengthDisplayingText.set({fill:'red'});
    textBox.text = textBox.text.slice(0, languageMaxLength );
    this.canvas.discardActiveObject();
    this.canvas.setActiveObject(textBox);
  } else {
    centerLengthDisplayingText.set({fill:'blue'});
    centerLengthDisplayingText.text = String(textBox.text.length + '/' + languageMaxLength);
  }
}

const limitRightTextAmount = (textBox) => {
  const numberMaxLength = 18;
  const rightLengthDisplayingText = this.canvas.getObjects().find((item) => item.name.includes('text_length_right'));
  if (textBox.text.length > languageMaxLength) {
    centerLengthDisplayingText.set({fill:'red'});
    textBox.text = textBox.text.slice(0, languageMaxLength );
    this.canvas.discardActiveObject();
    this.canvas.setActiveObject(textBox);
  } else {
    rightLengthDisplayingText.set({fill:'blue'});
    rightLengthDisplayingText.text = String(textBox.text.length + '/' + numberMaxLength);
  }
}
```

## <span style="color:#802548">_placeholder 기능 구현하기_</span>

- HTML의 경우, input에 placeholder 기능이 property로 존재한다.
- 따라서 별다른 노력 없이 구현이 가능하다.
- 하지만 canvas api에서는 Text object만이 존재하는데, 해당 속성이 없다.
- 따라서 어렵게 돌아가야 한다. text는 빈칸으로, 그 위에 placeholder 객체를 만들어두는 형태다.
- 일단 클릭하면 placeholder는 사라지고, text는 active에서 바로 editing 상태로 들어간다.
- 내용을 입력하지 않고 다른 text를 누르면 해당 text에는 placeholder text 객체를 다시 생성한다.
- placeholder기 때문에 위치값은 Text 객체와 동일하게 설정해준다.

```js
const leftTextPlaceHolderObj = new fabric.Text(leftDefaultText, {
  fontSize: bottomFontSize,
  fill: 'black',
  fontFamilty: "Malgun Gothic",
  name: 'text_placeholder_left',
  selectable:false,
})
leftTextPlaceHolderObj.set('left', leftTextBox.left);
leftTextPlaceHolderObj.set('top', leftTextBox.top);
leftTextPlaceHolderObj.set('width', leftTextBox.width);
leftTextPlaceHolderObj.set('height', leftTextBox.height);

const rightTextPlaceHolderObj = new fabric.Text(rightDefaultText, {
  fontSize: bottomFontSize,
  fill: 'black',
  fontFamilty: "Malgun Gothic",
  name: 'text_placeholder_right',
  selectable:false,
})
rightTextPlaceHolderObj.set('left', rightTextBox.left);
rightTextPlaceHolderObj.set('top', rightTextBox.top);
rightTextPlaceHolderObj.set('width', rightTextBox.width);
rightTextPlaceHolderObj.set('height', rightTextBox.height);

const leftTextPlaceHolderObj = new fabric.Text(leftDefaultText, {
  fontSize: bottomFontSize,
  fill: 'black',
  fontFamilty: "Malgun Gothic",
  name: 'text_placeholder_center',
  selectable:false,
})
centerTextPlaceHolderObj.set('left', centerTextBox.left);
centerTextPlaceHolderObj.set('top', centerTextBox.top);
centerTextPlaceHolderObj.set('width', centerTextBox.width);
centerTextPlaceHolderObj.set('height', centerTextBox.height);
```

- 역시 layer에는 추가할 필요가 없으니 레이어에서는 뺴준다.

```js
const LAYER_IGNORE = [
  "text_length_left",
  "text_length_right",
  "text_length_center",
  "text_placeholder_left",
  "text_placeholder_right",
  "text_placeholder_center",
]
```

- 이젠 event를 만들어주면 된다. 우선 placeholderObj를 클릭하는 순간, 사라지게 해야 한다.
- 그러면서 동시에 원래 text를 active가 아닌 바로 editing 상태로 돌입시킨다.

```js
this.canvas.on('mouse:down', (e) => {
  if (!e.target.name.includes('text_placeholder_')) {
    return;
  }

  const placeHolderObj = e.target;
  this.canvas.remove(placeHolderObj);

  if (e.target.name.includes("text_placeholder_left")) {
    const leftText = this.canvas.getObjects().find((item) => item.name.includes("text_left"));
    leftText.enterEditing();
  } else if (e.target.name.includes("text_placeholder_right")) {
    const rightText = this.canvas.getObjects().find((item) => item.name.includes("text_right"));
    rightText.enterEditing();
  } else (e.target.name.includes("text_placeholder_center")) {
    const centerText = this.canvas.getObjects().find((item) => item.name.includes("text_center"));
    centerText.enterEditing();
  }
})
```

- 이제 아무 입력을 하지 않고 다른 text를 선택했을 때, 다시 placeholder를 만든다.
- 혹시 처음부터 공백을 입력한 경우도 빈칸으로 간주하여 placeholder를 만들어놓는다.
- 클릭을 했을 때 placeholder가 사라지므로, placeholder가 없는지 확인한다. placeholder가 없을 때만 다시만들면 되기 때문이다.
- placeholder가 있는데 다시 만들면 계속 text가 겹쳐서 색이 진해진다. 뒤로가기, 앞으로가기 관리도 어려워진다.

```js
let placeholderArray = null;

$btnText.onclick = () => {
  const leftTextPlaceHolderObj = new fabric.Text(leftDefaultText, {
    fontSize: bottomFontSize,
    fill: 'black',
    fontFamilty: "Malgun Gothic",
    name: 'text_placeholder_left',
    selectable:false,
  })
  leftTextPlaceHolderObj.set('left', leftTextBox.left);
  leftTextPlaceHolderObj.set('top', leftTextBox.top);
  leftTextPlaceHolderObj.set('width', leftTextBox.width);
  leftTextPlaceHolderObj.set('height', leftTextBox.height);
  .
  .
  placeholderArray = [leftTextPlaceHolderObj, rightTextPlaceHolderObj, centerTextPlaceHolderObj]
}
```

- 왼쪽text를 눌렀을 때 오른쪽, 중앙 모두 검사해서 내용이 없으면 placeholder를 만든다.
- 마찬가지로 오른쪽 text를 눌렀을 때, 왼쪽과 중앙 모두 검사해서 내용이 없으면 placeholder를 만든다.
- 가운데도 마찬가지다.

```js

this.canvas.on('text:editing:entered', (e) => {
  const leftText = this.canvas.getObjects().find((item) => item.name.includes("text_left"));
  const rightText = this.canvas.getObjects().find((item) => item.name.includes("text_right"));
  const centerText = this.canvas.getObjects().find((item) => item.name.includes("text_center"));
  const leftPlaceholder = this.canvas.getObjects().find((item) => item.name.includes("text_placeholder_left"));
  const rightPlaceholder = this.canvas.getObjects().find((item) => item.name.includes("text_placeholder_right"));
  const centerPlaceholder = this.canvas.getObjects().find((item) => item.name.includes("text_placeholder_center"));

  if (e.target.name.include('text_left')) {

    if (!rightText.text.trim() && !rightPlaceholder) { 
      this.canvas.add(placeholderArray[1]);
    }

    if (!centerText.text.trim() && !centerPlaceholder) {
      this.canvas.add(placeholder[2]);
    }
  } else if (e.target.name.include('text_right')) {

    if (!leftText.text.trim() && !leftPlaceholder) {
      this.canvas.add(placeholderArray[0]);
    }

    if (!centerText.text.trim() && !centerPlaceholder) {
      this.canvas.add(placeholder[2]);
    }
  } else {

    if (!leftText.text.trim() && !leftPlaceholder) {
      this.canvas.add(placeholderArray[0]);
    }

    if (!rightText.text.trim() && !rightPlaceholder) {
      this.canvas.add(placeholder[1]);
    }
  }
})
```

- enter가 안되게도 구현해야 한다. 줄바꿈을 못하게 해달라고 했다.
- canvas api의 text object도 DOM이긴하지만, 개발자도구창엔 잡히지 않는 dom이다.
- 따라서 js로만 포착 가능하다. 거기다 canvas api에는 enter event를 따로 포착하는 게 불가능하다.
- document로만 포착해야함을 의미한다.
- 그런데 canvas api로 만든 text도 Dom의 기준에서는 textarea다. 따라서 tagName으로 포착한다.

```js
document.addEventListener('keydown', function() {
  if (e.key !=='Enter') {
    return;
  }

  if (e.target.tagName === 'TEXTAREA') {
    e.preventDefault();
  }

})
```


## <span style="color:#802548">_tempCanvas를 이용해 현재 canvas에서 필요없는 텍스트 길이, placeholder 지워서 서버에 저장하기_</span>

- text length 라던가, placeholder 등은 서버에 저장되는 카드 이미지에서는 지워야 한다.
- 따라서 현재 canvas가 아닌 tempcanvas를 활용해야 한다.
- tempcanvas는 현재 canvas와 동일한 크기와 높이를 지니게 만든다.

```js
const modalCanvas = new fabric.Canvas("confirmation-canvas", {
  width: modalWidth,
  height: modalHeight,
});

const tempCanvas = new fabric.Canvas("temp-canvas", {
  width: this.canvas.width,
  height: this.canvas.height
});
```

- canvas 자체를 데이터화해서 tempCanvas에 그려준다.
- 그려줄 때 LAYER_IGNORE에 있는 것들은 전부 remove하게 한다. 그럼 length text와 placeholder를 지워버린다.
- 그럼 지워진 canvas로 toDataURL()을 써서 base64Url을 만들 수 있다. 
- 해당 URL로 image를 만들어 confirm용으로 소비자에게 보여줄 canvas에 그린다.
- croppedImageData에는 tempCanvas의 zoom을 가져온다고 했는데, tempCanvas가 곧 원 canvas다.
  - 따라서 this.canvas로 써도 상관없다.

```js
const canvasJsonData = this.canvas.toObject(["name"])

tempCanvas.loadFromJSON(canvasJsonData, () => {
  // 이미지의 사이즈 설정은 this.canvas의 값으로 설정, tempCanvas의 object를 이미지 url화
  const deletedObject = tempCanvas.getObjects().filter((item) => LAYER_IGONORE.includes(item.name))

  deletedObject.forEach(item => {
      if(item.name == 'background') {
          return;
      }
      tempCanvas.remove(item);
  });

  const canvasWrap = getEl("#confirm-container #container");
  const _confirmCanvasElem = confirmCanvas.getElement();
  canvasWrap.append(_confirmCanvasElem)

  const background = new fabric.Rect({
      name: "background",
      left: 0,
      top: 44,
      width: this.canvas.width,
      height: this.canvas.height,
      fill: "#eee",
      stroke: "#000",
      strokeWidth: 0,
      selectable: false,
  });

  const mainObject = new fabric.Rect({
      name: "background",
      left: this.canvas.width / 2,
      top: this.canvas.height / 2 + 44,
      originX: "center",
      originY: "center",
      width: viewWidth,
      height: viewHeight,
      fill: "#fff",
      stroke: "#000",
      strokeWidth: 0,
      selectable: false,
  });

  const croppedImageData = tempCanvas.toDataURL({
      left:
          (mainObject.left - mainObject.width / 2) * tempCanvas.getZoom() +
          tempCanvas.viewportTransform[4],
      top:
          (mainObject.top - mainObject.height / 2) * tempCanvas.getZoom() +
          tempCanvas.viewportTransform[5],
      width: viewWidth * tempCanvas.getZoom(),
      height: viewHeight * tempCanvas.getZoom(),
      multiplier: 1 / tempCanvas.getZoom(),
  });

  fabric.Image.fromURL(croppedImageData, (img) => {
      // 이미지의 크기, 위치 등 설정
      img.set({
          left: tempCanvas.width / 2,
          top: tempCanvas.height / 2 + 44,
          originX: "center",
          originY: "center",
          scaleX: 1,
          scaleY: 1,
          selectable: false,
      });
      confirmCanvas.clear();

      // canvas에 이미지 추가
      confirmCanvas.add(background);
      confirmCanvas.add(img);
  });

  fabric.Image.fromURL(cardImageUrl, (img) => {
      img.set({
          left: tempCanvas.width / 2,
          top: tempCanvas.height / 2 + 44,
          width: viewWidth,
          height: viewHeight,
          originX: "center",
          originY: "center",
          selectable: false,
      });
      confirmCanvas.add(img)
  })
})
```

## <span style="color:#802548">_web의 text 생성 토글/ mobile의 text 생성 토글_</span>

- web의 구조는 텍스트를 누르면 tool이 헤더에 뜨는 구조다.
- 따라서 boolean 변수 하나로 전부 통제가 가능했다.

```js
let isTextCreated = false;
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
```

- 그러나 모바일의 UI는 다르다.
- text를 눌러도 확정 생성이 아니라, 한번더 확정 여부를 검사한다.

```js

```


## <span style="color:#802548">_이미지내에서만 cropzone 이동하게 하기_</span>

- 


```js

```


## <span style="color:#802548">_api 시간지연에 따른 모달창 띄우기_</span>
- api 시간지연이 15초 이상 지속되면 모달창을 띄우는 기능을 만들어보자.
- timer로 만들 setTimeout 함수를 만들어준다.

```js
let loadingTimer;

let objects = this.canvas.getObjects();
// 이미지 몇장인지 확인
let count = 0;

objects.forEach((obj) => {
    if (obj.name && obj.name.startsWith('image_')) {
        count++;
    }
});

// 15초 후에 팝업을 띄우는 타이머 설정
loadingTimer = setTimeout(() => openModal('modal09'), 15000);
```

- 응답이 15초 안에 오게끔 하려면 동기적으로 코드가 진행되게 해야 한다.
- 따라서 api는 async await로 구성한다.

```js
const removeBg = async() => {
    let loadingTimer;

    let objects = this.canvas.getObjects();
    // 이미지 몇장인지 확인
    let count = 0;

    objects.forEach((obj) => {
        if (obj.name && obj.name.startsWith('image_')) {
            count++;
        }
    });

    // 15초 후에 팝업을 띄우는 타이머 설정
    loadingTimer = setTimeout(() => openModal('modal09'), 15000);

    const resImg = await requestBackgroundRemove(imgObj.getSrc()); 
    clearTimeout(loadingTimer);
}
```

- 그런데 api에서 error가 나게 되면 catch를 하지 않으면 후속 코드가 진행되지 않는다.
- 따라서 실패 응답이 왔는데도 15초 뒤에 시간지연 모달창이 뜨게 된다.
- 따라서 try ~ catch로 감싸줘야 한다. 

```js
const removeBg = async() => {
    let loadingTimer;

    let objects = this.canvas.getObjects();
    // 이미지 몇장인지 확인
    let count = 0;

    objects.forEach((obj) => {
        if (obj.name && obj.name.startsWith('image_')) {
            count++;
        }
    });

    // 15초 후에 팝업을 띄우는 타이머 설정
    loadingTimer = setTimeout(() => openModal('modal09'), 15000);

    try {
        const resImg = await requestBackgroundRemove(imgObj.getSrc()); // 배경제거api 요청
        // 서버로부터 응답은 png형태
        const resImgSrc = 'data:image/png;base64,' + JSON.parse(resImg.response).img_str;

        imgObj.setSrc(resImgSrc, () => {
            this.canvas.requestRenderAll();
            getEl('#loadingBox').style.display = 'none';
            this.canvas.fire("object:modified", {target : imgObj})
        });
    } catch (error) {
        alert(error);
    }
    clearTimeout(loadingTimer);
}
```

## <span style="color:#802548">_promise로 감싸서 동기 코드로 만들기_</span>

- 서버에서 배경을 제거하고 blob 객체를 준다.
- 그런데 api는 await로 처리해놓고, 서버가 준 blob을 base64로 전환하는 function에는 await를 쓰지 않았다.
- 문제는 reader의 onload, readAsDataURL이 비동기함수라는 점이다.
- 그럼 서버에서 정상 응답을 줘도 절대로 제대로 이미지가 그려지지 않는다.
- 비동기라서 resImgSrc가 계산돼서 저장되기 전에 아래 코드가 실행되고, 그럼 undefined가 된다.
- 따라서 imgObj는 undefined를 setSrc하게 되니 오류가 나는 것이다.

```js
try {
    const resImg = await requestBackgroundRemove(imgObj.getSrc()); // 배경제거api 요청
    // 서버로부터 응답은 png형태
    const resImgSrc = blobToDataURL(resultBlob);
    console.log(resImgSrc);

    imgObj.setSrc(resImgSrc, () => {
        this.canvas.requestRenderAll();
        getEl('#loadingBox').style.display = 'none';

        // image 배경 제거시 object 수정 이벤트 발생
        this.canvas.fire("object:modified", {target : this.canvas})
        $removeAction01.checked = false;
        $removeAction02.checked = false;
        changeTool(TOOL.removebg, this.canvas)
    });

    imgObj.setSrc(resImgSrc, () => {
        this.canvas.requestRenderAll();
        getEl('#loadingBox').style.display = 'none';
        this.canvas.fire("object:modified", {target : imgObj})
    });
} catch (error) {
    alert(error);
}

function blobToDataURL(blob) {
    const reader = new FileReader();
    reader.onload = () => reader.result;
    reader.onerror = console.log('error');
    reader.readAsDataURL(blob);
}
```

- 이를 피하려면 동기로 만들어줘야 한다. await를 쓸수있게 똑같이 Promise로 감싸주어야 한다는 의미다.

```js
const resImg = await requestBackgroundRemove(imgObj.getSrc()); // 배경제거api 요청
// 서버로부터 응답은 png형태
const resImgSrc = await blobToDataURL(resultBlob);

function blobToDataURL(blob) {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = () => resolve(reader.result);
        reader.onerror = reject;
        reader.readAsDataURL(blob);
    });
}
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


- js

```js
const { x: objLeft, y: objTop } = obj.getPointByOrigin('left', 'top');
```