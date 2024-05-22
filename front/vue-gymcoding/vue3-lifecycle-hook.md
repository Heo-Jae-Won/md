- dom은 만들어진 이후가 setup()
- onMounted에서는 ref로 잡은 dom을 가져올 수 있다.
- 반면에 onBeforeMounted에서는 dom은 형성되었어도 mount가 되지 않아 null이다.

```js
<template>
    <div class="container py-4">
        <input ref="inputRef" type="text" value="hello world!" >
    </div>
</template>

<script>
import { onBeforeMount, onMounted, ref } from 'vue';

export default {
    setup() {
        const inputRef = ref(null);
        onBeforeMount(() => {
            console.log('onBeforeMount', inputRef.value); //null
        });

        onMounted(()=> {
            conosle.log('onMountd', inputRef.value); //<input type="text">
            conosle.log('onMountd', inputRef.value.value); //hello wolrd
        })

        return { inputRef };
    }
}

```



- 부모 = 자식 component 간 생성과 mount관계는 아래와 같다.
- 부모가 먼저 초기화되지만, 자식이 먼저 그려진다.

```
부모 생성
부모 mount 시작
자식 생성
자식 mount 시작
자식 mount 완료
부모 mount 완료
```


- onBeforeUpdate나 onUpdated는 반응형 변수들의 변화에만 반응한다.