
```js
//FancyButton.vue
<template>
    <button class="fancy-btn">
        <slot></slot>
    </button>
</template>

<script>
export default {
    setup() {
        return {};
    }
}
</script>

<style scoped>
.fancy-btn {
    color: #fff;
    background: linear-gradient(315deg, ...);
    border: none;
    .
    .
    .
}
</style>

//
<FancyButton>
    Click!!
</FancyButton>
```

- 그럼 실제로 Click!!으로 나온다.
- FancyButton component에 text를 넣지 않아도 된다!

```js
<FancyButton>
    Click!!<span style="color: red">@@@</span>
</FancyButton>
```

- slot에 값을 안 넣는 경우, default content를 넣을 수도 있다.

```js
//parent component
<FancyButton></FancyButton>

//FancyButton.vue
<template>
    <button class="fancy-btn">
        <slot>deafult content</slot>
    </button>
</template>

```


- named slots 기능도 있다.
- 이름을 주었으니 이름에 맞게 content가 들어가게 된다.
- 이름이 없는 것은 default로 선언된 곳으로 간다.

```js
//BaseCard.vue
<template>
    <article>
        <div>
            <slot name="header"></slot>
        </div>
        <div>
            <slot></slot>
        <div>
            <slot name="footer"></slot>
        </div>
    </article>
</template>


//named로 header와 footer에 맞게 넣기
<template>
    <BaseCard>
        <template v-slot:header>제목</template>
        <template v-slot:default>안녕하세요</template>
        <template v-slot:footer>푸터</template>
    </template>
</BaseCard>
```

- v-slot은 #으로 대체할 수 있다.

```js
<template>
    <BaseCard>
        <template #header>제목</template>
        <template #default>안녕하세요</template>
        <template #footer>푸터</template>
    </template>
</BaseCard>
```

- reactivity 변수를 이용할 수도 있다.
- header자리에만 제목입니다.로 나오게 된다.
- 만약 아래 속성을 vue-dev-tools에서 header가 아닌 footer로 변경하면?
- header가 아닌 footer에 제목입니다 text가 뜨게 된다.

```js
<template>
    <BaseCard>
        <template #[slotArgs]>제목입니다</template>
</BaseCard>

<script>
setup() {
    const slotArgs = ref('header');
    
    return {slotArgs};
}

</script>
```

```js
<template>
    <BaseCard>
        <template #header>제목</template>
        <template #default>안녕하세요</template>
        <template #footer>푸터</template>
    </template>
</BaseCard>
```

- 기본값을 부모 component에서 지정할 수도 있다.

```js
//ParentComponent.vue
<template>
    <BaseCard>
        <template #default="obj">
            제목입니다
            {{obj}}
        </template>
    </template>
</BaseCard>

//ChildComponent.vue
<template>
    <div class="card-body">
        <slot :child-message="childMessage" hello-message="안녕하세요">#Body</slot>
    </div>
</template>

<script>
export default {
    setup() {
        const childMessage = ref('자식 컴포넌트 메시지');
        return { childMessage};
    }
}
</script>
```

- 넘긴 obj에는 아래와 같이 담겨있다.

```
{
    "childMessage":"자식 컴포넌트 메시지",
    "helloMessage":"안녕하세요"
}
```

- default message를 출력하는 경우에는 template을 쓰지 않는다. 
- 안에다가 바로 v-slot을 박을 수 있다.

```js
//ParentComponent.vue
<template>
    <BaseCard v-slot="{fancyMessage}">
        {{fancyMessage}}
    </BaseCard>
</template>

//ChildComponent.vue
<template>
    <div class="card-body">
        <slot :fancy-message="fancyMessage">#Body</slot>
    </div>
</template>
```


- v-if를 사용하면 footer slot이 parent Component에 있는 경우에만 rendering할 수도 있다.
- $slots를 사용해서 존재 여부를 확인할 수도 있고, 이를 js 단에서 computed로 확인할 수도 있따.
- 아래는 default밖에 없어서 default만 뜨게 된다.

```js
//ParentComponent.vue
<template>
    <BaseCard v-slot="{fancyMessage}">
        {{fancyMessage}}
    </BaseCard>
</template>

//ChildComponent
<template>
    <div class="card">
        <div v-if="$slots.header" class="card-header">
            <slot name="header" header-message="헤더 메시지"></slot>
        </div>
        <div v-if="$slots.default" class="card-body">
            <slot :child-message="childMessage" hello-message="안녕하세요!"></slot>
        </div>
        <div v-if="hasFooter" class="card-footer test-muted">
            <slot name="footer" footer-message="푸터 메시지"></slot>
        </div>
    </div>
</template>

<script>
import { computed } from '@vue/reactivity';
import { ref } from 'vue';

export default {
    setup(props, { slots }) {
        const childMessage = ref('자식 컴포넌트 메시지');
        const hasFooter = computed(() => !!slots.footer);

        return { childMessage, hasFooter };
    }
}
</script>
```

- default가 아니라 전부 규정해주면 전부 HTML에 드러난다.

```js
//ParentComponent.vue
<template>
    <BaseCard>
        <template #default>{{childMessage}}</template>
        <template #header>{{headerMessage}}</template>
        <template #footer>{{footerMessage}}</template>
    </BaseCard>
</template>

```



```js

```