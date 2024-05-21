
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
    </template>
</BaseCard>

<script>
setup() {
    const slotArgs = ref('header');
    
    return {slotArgs};
}

</script>
```



