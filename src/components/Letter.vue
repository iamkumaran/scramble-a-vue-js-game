<template>
  <li v-bind:style="{ left: ((index * 100) + (index * 15)) + 'px' }"><span>{{data}}</span></li>
</template>

<script>
export default {
  name: 'letter',
  props: ['data', 'index']
}
</script>
