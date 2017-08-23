<template>
  <div id="user-progress">
      <div class="levels">LEVEL: <span>{{level}}</span></div>
      <div class="scores">SCORE: <span>{{score}}</span></div>
    </div>
</template>

<script>
export default {
  name: 'scores-levels',
  computed: {
    score: function () {
      return this.$store.state.score;
    },
    level: function () {
      return this.$store.state.level;
    }
  }
}
</script>
