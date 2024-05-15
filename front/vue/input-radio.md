## <span style="color:#802548">_1. vue의 input은 v-model_</span>
- 아래와 같이 input이 만들어져 있다고 해보자.
- confirm을 취소하는 경우는 check를 막아버리고 싶은데, checked로는 그게 안된다.

```html
<template>
    <input type="radio" :checked="kind === 'math' " value="math" @change="selectSubject">
    <input type="radio" :checked="kind === 'kor' " value="kor" @change="selectSubject">
</template>

<script>
data() {
    kind: "",
 },
methods: {
    selectSubejct(event) {
        const oldValue = this.kind;
        const newValue = event.target.value;
        if (oldValue !== newValue) {
            const result = confrim("변경하시겠습니까?");
            if (!result) {
                this.kind = oldValue;
                return ;
            } else {
                this.kind = newValue;
            }
        }
    }
}
</script>
```

- vue를 쓰려면 역시 v-model을 써야 한다.
  - v-model을 쓰면 click event가 일어나는 순간, value가 바뀌게 된다.
  - input에 checked로 바뀌는 event를 막는건 change event에선 불가능하다.
  - input에 click event에서 이미 checked가 일어나고, 그 다음 change event에서 click된 것을 막는 게 불가능하다.
  - 그래서 change가 아닌 click에서 event를 막아야 한다.
  - 정확히는 click event를 감지하되, event를 없던 것처럼 되돌리는 것이다.


```html
<input
  type="radio"
  value="math"
  @click="clickSubject"
  v-model="selectedSubject"
/>수학
<input
  type="radio"
  value="kor"
  @click="clickSubject"
  v-model="selectedSubject"
/>국어
<input
  type="radio"
  value="eng"
  @click="clickSubject"
  v-model="selectedSubject"
/>영어
```

```js
 data() {
    selectedSubject: "",
 },
 methods: {
    clickSubject(event) {
      if (this.selectedSubject === event.target.value || !this.selectedSubject) {
        return;
      }

      const result = confirm("변경하시겠습니까?");
      if (!result) {
          event.preventDefault();
        } else {
            this.selectedSubject = event.target.value
        }
        console.log(this.selectedSubject);
    }
 }
```
