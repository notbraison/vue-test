<template>
  <div class="hello-world">{{ msg }}</div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';

export default defineComponent({
  name: 'HelloWorld',
  props: {
    msg: {
      type: String,
      required: true,
    },
  },
});
</script>

<style scoped>
.hello-world { font-size: 14px; }
</style>
