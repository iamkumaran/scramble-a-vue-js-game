<template>
  <div id="game-menu" class="game-container">
      <div class="site-logo"><a href="http://iamkumaran.github.io/" target="_blank"><img src="http://iamkumaran.github.io/img/logo.png" title="Back To My Github Page"/></a></div>
      <div class="logo"><img src="../assets/scramble-logo.png" title="logo" alt="logo"/></div>
      <ol class="options">
        <li v-if="!$store.state.isGameStarted"><button class="btn" v-on:click="initGame">Play</button></li>
        <li v-if="isPaused"><button class="btn" v-on:click="resumeGame">Resume</button></li>
        <li><a class="btn" href="https://github.com/iamkumaran/scramble-a-vue-js-game" target="_blank">Github</a></li>

        <li v-if="isPaused"><button class="btn" v-on:click="quitGame">Quit Game</button></li>
        <li><div class="addthis_inline_share_toolbox"></div></li>
        <li>
          <div class="fb-like"
            data-href="http://iamkumaran.github.io/scramble-a-vue-js-game/"
            data-layout="standard"
            data-action="like"
            data-show-faces="true"></div>
        </li>

        <li>Leave a Comment</li>
      </ol>

      <vue-disqus shortname="iamkumaran" url="http://iamkumaran.github.io/scramble-a-vue-js-game/" identifier="Scramble - A Vue.js Game Page"></vue-disqus>
    </div>
</template>

<script>
import VueDisqus from '../components/VueDisqus'

export default {
  name: 'game-menu',
  components: {
    VueDisqus
  },
  computed: {
    isPaused: function () {
      return this.$store.state.isGamePaused;
    }
  },
  methods: {
    initGame: function () {
      this.$emit('initGame');
    },
    resumeGame: function () {
      this.$store.dispatch('pauseGame', this.$store.state.isGamePaused);
    },
    quitGame: function () {
      this.$store.dispatch('resetGame');
    }
  },
  mounted: function () {
    if (typeof window.addthis !== 'undefined' && typeof window.addthis.layers !== 'undefined' && typeof window.addthis.layers.refresh !== 'undefined') {
      window.addthis.layers.refresh();
    }

    if (typeof window.FB !== 'undefined' && typeof window.FB.XFBML !== 'undefined' && typeof window.FB.XFBML.parse !== 'undefined') {
      window.FB.XFBML.parse();
    }
  }
}
</script>
