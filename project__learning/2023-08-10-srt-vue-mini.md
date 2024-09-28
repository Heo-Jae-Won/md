## <span style="color:#802548">_jenkins github 연동_</span>

- button은 component를 

```js
<template>
  <div>
    <v-btn v-bind="$attrs">{{ message }}</v-btn>
  </div>
</template>

<script setup lang="ts">
defineProps<{
  message:string
}>();
</script>

<style scoped>

</style>
```

```js
<template>
  <div>
    <AppButton type="button" color="blue lighten-1 text-capitalize" depressed large block dark class="mb-3" @click.stop="$emit('save')" :message="'회원가입'"></AppButton>
    <AppButton type="button" color="blue lighten-1 text-capitalize" depressed large block dark class="mb-3" @click.stop="$emit('back')" :message="'뒤로가기'"></AppButton>
  </div>
</template>

<script setup lang="ts">
defineEmits(['save','back'])
</script>

<style scoped>

</style>
```

```js
<template>
  <v-main class="blue-grey lighten-4">
    <v-container class="mt-5" style="max-width: 700px" fill-height>
      <v-card>
        <div class="pa-15">
          <h1 style="text-align: center" class="mb-10">
          <slot name="header"></slot>
        </h1>
          <slot name="body"></slot>
        </div>
      </v-card>
    </v-container>
  </v-main>
</template>

<script setup lang="ts"></script>

<style scoped></style>
```