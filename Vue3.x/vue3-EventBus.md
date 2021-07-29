### vue3.x完全移除了 $on、$off 和 $once 方法。
+ Event bus 模式可以被替换为实现了事件触发器接口的外部库，例如 mitt 或 tiny-emitter
##### 在main.js全局注册事件
```javascript
import { createApp } from 'vue'
import App from './App.vue'
import Emitter from 'tiny-emitter'
const emitter = new Emitter()

const app = createApp(App)
// console.log("vue:" + app)
app.config.globalProperties.emitter = emitter
```
##### brother-one.vue
```javascript
<template>
  <div>
    <button @click="sendReq">事件总线</button>
  </div>
</template>
<script>
export default {
  name: "send",
  components: {},
  data() {
    return {};
  },
  methods: {
    sendReq() {
      this.emitter.emit("changeLoading", true);
    },
  },
};
</script>
```
##### brother-two.vue
```javascript
<template>
  <div>
  </div>
</template>
<script>
export default {
  name: "",
  components: {},
  data() {
    return {};
  },
  mounted() {
    this.emitter.on("changeLoading", (news) => {
      // this.isShowLoding = news;
      if (news) {
        console.log("收到over");
      }
    });
  },
  methods: {},
};
</script>
```
##### 父组件
```javascript
<template>
  <div>
    <A></A>
    <B></B>
  </div>
</template>
<script>
import A from "../components/brother-one.vue";
import B from "../components/brother-two.vue";
export default {
  name: "",
  components: { A, B },
  data() {
    return {};
  },
  methods: {},
};
</script>
```
