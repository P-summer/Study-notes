```javascript
// 子组件emits.vue  如果事件不是在根元素，可以不同emits声明，如果事件在根元素，不用emits声明会有警告
// 在 setup 函数或者methods里面发送emit事件效果一样
<template>
  <button @click="handleClick">点击emit-click事件</button>
  <button @click="handleOpen">点击emit-open事件</button>
</template>

<script>
export default {
  name: "emits-test",
  props: {},
  emits: {
    click: null, // 没有验证函数
    // 带有验证函数
    open: (value) => {
      if (typeof value === "string") {
        return true;
      } else {
        return false;
      }
    },
  },
  data() {
    return {};
  },
  // props  context:{ attrs, slots, emit }   执行 setup 时，组件实例尚未被创建。
  /* setup(props, { emit }) {
    const handleClick = function () {
      emit("click");
    };
    const handleOpen = function () {
      emit("open", 1);
    };
    return {
      handleClick,
      handleOpen,
    };
  }, */
  methods: {
    handleClick() {
      this.$emit("click", "click");
    },
    handleOpen() {
      this.$emit("open", 1);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
```
```javascript
// 父组件receiver.vue
<template>
  <Emits
    @click="onClick"
    @open="onOpen"
  ></Emits>
</template>
<script>
import Emits from "@/components/emits.vue";
export default {
  name: "receiver",
  components: {
    Emits,
  },
  data() {
    return {};
  },
  methods: {
    onClick() {
      console.log("click me!");
    },
    onOpen() {
      console.log("open me!");
    },
  },
};
</script>
```
