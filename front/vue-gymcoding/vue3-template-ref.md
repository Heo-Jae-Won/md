- template ref는 dom에 대한 직접 접근을 지양하기 위해 생겨났다.
- input.value는 처음 DOM을 그릴 때는 null이다.
- 따라서 mount가 끝나고 input 값이 생길 때 넣게 v-if를 추가한다.

```html
<templte>
    <div class="container py-4">
        <input ref="input" type="text" />
        <p> {{ input }} </p> <!--[object HTMLInputElment]-->
        <p v-if="input"> {{ input.value }}, {{ $refs.input.value }}, {{ $refs.input. === input}} </p>
    </div> 
</template>

<script>
import { ref } from 'vue';
export default {
    setup() {
        const input = ref(null);
        return { input,  };
    }
}
</script>
```

- dom에 ref를 박아두고 v - for를 돌리면 mount 이후에 ref 변수에 해당 값들이 들어온다.
- 따라서 onMounted에 itemsRef를 forEach로 돌리면 해당 내용을 확인해볼 수 있다.

```html
<templte>
    <div class="container py-4">
        <input ref="input" type="text" />
        <p> {{ input }} </p> <!--[object HTMLInputElment]-->
        <p v-if="input"> {{ input.value }}, {{ $refs.input.value }}, {{ $refs.input. === input}} </p>
        <hr>
        <ul>
            <li v-for="fruit in fruits" :key="fruit" ref="itemRefs">{{fruits}}</li>
        </ul>
    </div> 
</template>

<script>
import { ref } from 'vue';
export default {
    setup() {
        const input = ref(null);
        onMounted(() => {
            input.value.value = 'Hello World';
            console.log('onMounted: ', input.value);

            itemsRefs.value.forEach(item => console.log('item: ', item.textContent));
            /**
             * item: 사과
             * item: 딸기
             * item: 포도
             */
        })

        const fruits = ref(['사과','딸기','포도']);
        const itemRefs = ref([]);

        return { input, fruits, itemRefs };
    }
}
</script>
```
