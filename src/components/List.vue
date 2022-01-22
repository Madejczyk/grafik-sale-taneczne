<template>
    <li>{{ item.text }}</li>
</template>

<script>
export default {
  name: 'List',
  props: ['item']
}
</script>