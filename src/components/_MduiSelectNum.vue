<template>
  <select class="mdui-select" mdui-select :value="value" @change="$emit('change', parseInt($event.target.value))">
    <option v-for="opt of options" :key="`opt-${opt}`" :value="opt">{{opt}}</option>
  </select>
</template>

<script>
export default {
  name: 'mdui-select-num',
  model: {
    event: 'change',
  },
  props: {
    value: [Number, String],
    options: Array,
  },
};
</script>
