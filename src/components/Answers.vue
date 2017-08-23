<template>
  <div id="answers-list">
    <div>
      <div v-for="(opt, i) in answersList" class="answers">
        <span v-if="userAnswerList.indexOf(opt) !== -1" v-for="data in opt" class="highlight">{{data}}</span>
        <span v-if="userAnswerList.indexOf(opt) === -1" v-for="data in opt">&nbsp;</span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'letter',
  computed: {
    answersList: function () {
      return this.$store.state.gameData.c.opt;
    },
    userAnswerList: function () {
      return this.$store.state.gameData.c.userAnswers;
    }
  }
}
</script>
