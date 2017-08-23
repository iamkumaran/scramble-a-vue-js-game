<template>
  <div id="controls">
    <button class="sub-btn" v-on:click="submitAction" :disabled="enableSubmit===false">Submit</button>
    <button class="shuffle-btn" v-on:click="shuffleQuestion">Shuffle</button>
    <button class="last-word" v-on:click="lastWord" :disabled="enableLastWord===false">Last Word</button>
    <button class="clear-btn" v-on:click="clearAction">Clear</button>
  </div>
</template>

<script>
import Velocity from 'velocity-animate'

export default {
  name: 'controls',
  data: function () {
    return {
      isShuffling: false
    };
  },
  computed: {
    enableSubmit: function () {
      return this.$store.state.dropArea.join('').length > 2;
    },
    enableLastWord: function () {
      return !!this.$store.state.gameData.c.lastword;
    }
  },

  methods: {
    submitAction: function () {
      var word = this.$store.state.dropArea.join('');
      var i = this.$store.state.gameData.c.opt.indexOf(word);
      var isAlreadyFound = this.$store.state.gameData.c.userAnswers.indexOf(word);
      if (i !== -1 && isAlreadyFound === -1) {
        this.$store.dispatch('validWord', {value: word});
        this.$store.dispatch('lastWord', {value: word});
        for (var j = 0; j < this.$store.state.dropArea.length; j++) {
          this.$emit('bringDownAll', document.querySelector('#drop-area li:nth-child(' + (j + 1) + ')'), 'letterDown', this.$store.state.dropArea[j]);
        }

        this.$store.dispatch('updateScore', 100 * (this.$store.state.dropArea.length + 1));
      } else {
        if (isAlreadyFound !== -1) {
          this.$emit('setError', this.$store.state.error.exist);
        } else {
          this.$emit('setError', this.$store.state.error.invalid);
        }
        this.clearAction();
      }
    },
    lastWord: function () {
      var arr = this.$store.state.gameData.c.lastword.split('');
      for (var i = 0; i < arr.length; i++) {
        this.$emit('bringDownAll', document.querySelector('#q-area li[data-id=' + arr[i] + ']'), 'letterUp', arr[i]);
      }
    },
    shuffleQuestion: function () {
      if (this.isShuffling) return false;
      var q = this.$store.state.gameData.c.ques;
      var arr = q.slice();
      arr = this.shuffle(arr);
      var li = document.querySelectorAll('#q-area li');
      this.isShuffling = true;
      var $this = this;
      for (var i = 0; i < q.length; i++) {
        var index = arr.indexOf(q[i]);
        Velocity(li[i], {

          left: ((index * 100) + (index * 15)) + 'px'

        }, {
          duration: 250,
          complete: function (elements) {
            $this.isShuffling = false;
          }
        });
      }
    },
    shuffle: function (array) {
      var tmp;
      var current;
      var top = array.length;
      if (top) {
        while (--top) {
          current = Math.floor(Math.random() * (top + 1));
          tmp = array[current];
          array[current] = array[top];
          array[top] = tmp;
        }
      }
      return array;
    },
    clearAction: function () {
      this.$emit('clearAction');
    }
  }
}
</script>
