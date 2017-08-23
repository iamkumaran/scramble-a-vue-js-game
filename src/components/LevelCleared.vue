<template>
  <div class="level-cleared">
      <div class="logo"><img src="../assets/scramble-logo.png" title="logo" alt="logo"/></div>
      <p>Level {{$store.state.level}} Cleared.</p>
      <!--<p>You have found all {{$store.state.gameData.c.opt.length}} possible words.</p>-->
      <p><span class="highlight">{{$store.state.bonusScore}}</span> bonus points</p>
      <p><button class="btn" v-on:click="nextLevel">Next Level</button></p>
    </div>
</template>

<script>
export default {
  name: 'level-cleared',
  methods: {
    nextLevel: function () {
      this.$emit('nextLevel');
    }
  }
}
</script>
