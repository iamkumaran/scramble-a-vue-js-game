<template>
  <div id="gameApp">
    <section>
        <game-menu v-on:initGame="nextLevel" v-if="!isGameStarted || isPaused"/>

        <div class="game-container" v-if="isGameStarted" v-show="!isPaused">
          <div class="site-logo"><a href="http://iamkumaran.github.io/" target="_blank"><img src="http://iamkumaran.github.io/img/logo.png" title="Back To My Github Page"/></a></div>
          <button class="main-menu-btn btn" v-on:click="pauseGame">Main Menu</button>
          <scores-levels />

          <level-cleared v-if="$store.state.isAllWordsFound" v-on:nextLevel="nextLevel"/>

          <div v-if="!$store.state.isAllWordsFound">
            <transition name="slide-fade">
              <p class="error" v-if="errorMessage">{{errorMessage}}</p>
            </transition>
            <ul id="drop-area">
              <letter v-for="i in gameData.c.ques.length" v-bind:data="dropAreaData(i)" v-bind:index="i-1" v-bind:key="i-1" class="inactive" v-on:click.native="letterDown($event, dropArea[i-1])"></letter>
            </ul>
            <ul id="q-area">
              <letter v-for="(data, i) in gameData.c.ques" v-bind:data="data" v-bind:index="i" v-bind:key="i" v-on:click.native="letterUp($event, data)" :data-id="data"></letter>
            </ul>
            <ul id="cloners"></ul>
            <controls v-on:bringDownAll="letterAction" v-on:clearAction="clearAction" v-on:setError="setError"></controls>

            <transition name="slide-fade"><p class="next-level-button" v-if="$store.state.showNextLevel">Skip this level <button class="btn" v-on:click="nextLevel">Next Level</button></p></transition>
          </div>
          <answers></answers>

          <div class="how-to-play" v-if="showHowToPlayModal === true">
            <h3 style="text-align:center">How To Play</h3>
            <p>There are 5 jumbled letters. Arrange the letters to form possible WORDS.</p>
            <p>Click on YELLOW balls to move letters up to form WORDS. <b>&lt;Submit&gt;</b> button will be enabled once there are 3 letters at top.</p>
            <p>At-least one 5 letter word is required to go next level.</p>
            <p>Find all words to earn bonus points {{$store.state.bonusScore}}.</p>

            <p style="text-align:center"><button class="btn" v-on:click="showHowToPlayModal = false">Start</button></p>
          </div>
        </div>
      </section>
  </div>
</template>

<script>
import Velocity from 'velocity-animate'
import Letter from './components/Letter'
import GameMenu from './components/GameMenu'
import ScoresLevels from './components/ScoresLevels'
import LevelCleared from './components/LevelCleared'
import Controls from './components/Controls'
import Answers from './components/Answers'

export default {
  name: 'app',
  components: {
    Letter,
    GameMenu,
    ScoresLevels,
    LevelCleared,
    Controls,
    Answers
  },
  data () {
    return {
      errorMessage: '',
      showHowToPlayModal: false
    }
  },
  computed: {
    gameData: function () {
      return this.$store.state.gameData;
    },
    dropArea: function () {
      return this.$store.state.dropArea;
    },
    score: function () {
      return this.$store.state.score;
    },
    level: function () {
      return this.$store.state.level;
    },
    isGameStarted: function () {
      return this.$store.state.isGameStarted;
    },
    isPaused: function () {
      return this.$store.state.isGamePaused;
    }
  },
  methods: {
    nextLevel: function () {
      this.clearAction();
      var data = JSON.parse(this.decodeContent(this.$store.state.fetchedData));
      var i = this.getRandomNumber(0, data.length - 1);
      data[i].ques = this.shuffle(data[i].ques);
      data[i].userAnswers = [];
      data[i].lastword = '';

      this.$store.dispatch('nextLevel', data[i]);

      // show How to play modal
      this.showHowToPlayModal = !!(this.$store.state.level === 1);
    },
    pauseGame: function () {
      this.$store.dispatch('pauseGame', this.$store.state.isGamePaused);
    },
    decodeContent: function (str) {
      // Going backwards: from bytestream, to percent-encoding, to original string.
      return decodeURIComponent(atob(str).split('').map(function (c) {
        return '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2);
      }).join(''));
    },
    getRandomNumber: function (min, max) {
      return Math.floor(Math.random() * (max - min + 1) + min);
    },
    shuffle: function (arr) {
      var currIndex = arr.length;
      var tempValue;
      var randomIndex;
      while (currIndex !== 0) {
        randomIndex = Math.floor(Math.random() * currIndex);
        currIndex -= 1;
        tempValue = arr[currIndex];
        arr[currIndex] = arr[randomIndex];
        arr[randomIndex] = tempValue;
      }
      return arr;
    },
    setError: function (message) {
      this.errorMessage = message;
      setTimeout(function () { this.errorMessage = '' }.bind(this), 2000);
    },
    dropAreaData: function (n) {
      return this.dropArea.length ? this.dropArea[n - 1] : ''
    },
    letterUp: function (e, val) {
      e.stopPropagation();
      if (this.dropArea.indexOf(val) > -1) return false;

      // clone element
      var currElem = e.currentTarget;
      this.letterAction(currElem, 'letterUp', val);
    },
    letterDown: function (e, val) {
      if (!val) return false;
      e.stopPropagation();

      var currElem = e.currentTarget;

      this.letterAction(currElem, 'letterDown', val);
    },
    letterAction: function (currElem, type, data) {
      if (type === 'letterUp') { if (this.dropArea.indexOf(data) > -1) return false; }

      if (!data.trim()) return false;
      if (currElem.classList.contains('animating')) return false;

      var cloneLI = currElem.cloneNode(true);
      cloneLI.style.top = (type === 'letterUp') ? '200px' : '50px';
      var clonersElem = document.querySelector('#cloners');
      clonersElem.appendChild(cloneLI);

      currElem.classList.add('animating');
      currElem.classList.add('inactive');

      var i;
      var letterElem;
      var liStyles;
      if (type === 'letterUp') {
        if (this.dropArea.indexOf('') !== -1) {
          i = this.dropArea.indexOf('') + 1;
          // console.log('==>', i);
        } else {
          i = this.dropArea.length + 1;
        }
        letterElem = document.querySelector('#drop-area li:nth-child(' + i + ')');
        letterElem.classList.add('animating');
        liStyles = window.getComputedStyle(letterElem);
      } else {
        i = this.gameData.c.ques.indexOf(data) + 1;
        letterElem = document.querySelector('#q-area li:nth-child(' + i + ')');
        letterElem.classList.add('animating');
        liStyles = window.getComputedStyle(letterElem);
      }

      this.$store.dispatch(type, { value: data });

      var $this = this;
      Velocity(cloneLI, {
        top: liStyles.top,
        left: liStyles.left

      }, {
        duration: 250,
        complete: function (elements) {
          // $this.$store.dispatch(type, {value:data});

          clonersElem.removeChild(elements[0]);
          currElem.classList.remove('animating');

          letterElem.classList.remove('animating');
          letterElem.classList.remove('inactive');

          $this.arrayCleanUp();
        }
      });
    },
    clearAction: function () {
      var ln = this.$store.state.dropArea.length;
      if (!ln) return false;
      for (var i = 0; i < ln; i++) {
        if (!this.$store.state.dropArea[i]) continue;
        this.letterAction(document.querySelector('#drop-area li:nth-child(' + (i + 1) + ')'), 'letterDown', this.$store.state.dropArea[i]);
      }
    },
    arrayCleanUp: function () {
      if (!this.dropArea.length) return false;
      if (this.dropArea[this.dropArea.length - 1] === '') {
        this.$store.dispatch('arrayCleanUpDropArea');
        this.arrayCleanUp();
      }
    }
  }
}
</script>

<style>
body{font-family: sans-serif;}
.logo{text-align: center;}
.site-logo{position:absolute;top: -14px;z-index:1;}
.site-logo img{width:50px;height:50px;}
.how-to-play{position: absolute;z-index: 2;width: 99%;height: 380px;background: rgba(255, 255, 255, 0.87);border-radius: 13px;top: 45px;padding: 13px;box-sizing: border-box;font-size: 18px;}
#game-menu .site-logo{top: -6px;}
#gameApp .game-container{
  position: relative;
  width: 570px;
  margin: auto;
}
#controls{width: 570px;text-align:center;}
#answers-list{position: absolute;top:470px;width:115%;}
#answers-list{}
#answers-list .answers{padding: 3px 5px;font-size: 1em;display: inline-block;width: 150px;}
#answers-list span{display: inline-block;border: 1px solid grey;margin: 1px 3px;width:20px;height:20px;text-align: center;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  box-sizing: border-box;
  background: rgba(255, 239, 213, 0.67);
}

#answers-list span.highlight{background: #e82828;color: #fff;}

ul{
  list-style: none;
}


ul li {
  display: inline-block;
  width: 100px;
  height: 100px;
  margin: 0;
  border-radius: 50%;
  position: absolute;
  /*background: radial-gradient(circle at 50% 120%, #81e8f6, #76deef 10%, #055194 80%, #062745 100%);*/
  background: #f5fd2d;

  display: flex;
  align-items: center;
  justify-content: center;

  opacity: 1;

  font-weight: bold;
  font-size: 4em;
  border: 1px solid #aab7ad;
  cursor: pointer;
}
ul li:after {
  content: "";
  position: absolute;
  top: 1%;
  left: 5%;
  width: 90%;
  height: 90%;
  border-radius: 50%;
  background: radial-gradient(circle at 50% 0px, #ffffff, rgba(255, 255, 255, 0) 58%);
  -webkit-filter: blur(5px);
  z-index: 2;
}

ul li.inactive, ul li.animating, #drop-area li.animating {

  background: radial-gradient(circle at 50% 40%, #fcfcfc, #efeff1 66%, #9b5050 100%);
  cursor: not-allowed;

}
ul li.inactive:after, ul li.animating:after {

  top: 72%;
  left: 62%;
  width: 100%;
  height: 100%;

  background: radial-gradient(circle at 50% 50%, rgba(255, 255, 255, 0.8), rgba(255, 255, 255, 0.8) 14%, rgba(255, 255, 255, 0) 24%);
  -webkit-transform: translateX(-80px) translateY(-90px) skewX(-20deg);
  -moz-transform: translateX(-80px) translateY(-90px) skewX(-20deg);
  -ms-transform: translateX(-80px) translateY(-90px) skewX(-20deg);
  -o-transform: translateX(-80px) translateY(-90px) skewX(-20deg);
  transform: translateX(-80px) translateY(-90px) skewX(-20deg);
}


#drop-area li.animating span{visibility: hidden;}







/*ul li.inactive, ul li.animating{
  background: radial-gradient(circle at 25px 25px, #ffffff, #5e89b7);
}*/

#cloners li.inactive{background: radial-gradient(circle at 50% 40%, #fcfcfc, #efeff1 66%, #9b5050 100%);}

#drop-area li{
  top: 50px;
}
#q-area li{top: 200px;}

#controls{position:absolute;top:335px;}
#controls button{padding:30px;cursor:pointer;}

#user-progress{font-size:22px;}
#user-progress span{font-weight: bold;}
.levels{float:right;}
.scores{float:left;margin-left: 80px;}


/*Buttons*/
#controls button {
  position: relative;
  cursor: pointer;
  padding: 12px 16px;
  overflow: hidden;
  outline: none;
  border: 0 solid rgba(0,0,0,1);
  border-radius: 10px;
  font: normal normal bold 1.6em/normal "Syncopate", Helvetica, sans-serif;
  color: #ecf0f1;
  background: linear-gradient(180deg, #ff8583 0, #ff5350 100%);
  box-shadow: 0 10px 0 0 rgba(178,49,49,1) ;
  text-shadow: 0 0 0 rgb(196,80,78) , 1px 1px 0 rgb(196,80,78) , 2px 2px 0 rgb(196,80,78) , 3px 3px 0 rgb(196,80,78) , 4px 4px 0 rgb(196,80,78) , 5px 5px 0 rgb(196,80,78) , 6px 6px 0 rgb(196,80,78) , 7px 7px 0 rgb(196,80,78) , 8px 8px 0 rgb(196,80,78) , 9px 9px 0 rgb(196,80,78) , 10px 10px 0 rgb(196,80,78) , 11px 11px 0 rgb(196,80,78) , 12px 12px 0 rgb(196,80,78) , 13px 13px 0 rgb(196,80,78) , 14px 14px 0 rgb(196,80,78) , 15px 15px 0 rgb(196,80,78) , 16px 16px 0 rgb(196,80,78) , 17px 17px 0 rgb(196,80,78) , 18px 18px 0 rgb(196,80,78) , 19px 19px 0 rgb(196,80,78) , 20px 20px 0 rgb(196,80,78) , 21px 21px 0 rgb(196,80,78) , 22px 22px 0 rgb(196,80,78) , 23px 23px 0 rgb(196,80,78) , 24px 24px 0 rgb(196,80,78) , 25px 25px 0 rgb(196,80,78) , 26px 26px 0 rgb(196,80,78) , 27px 27px 0 rgb(196,80,78) , 28px 28px 0 rgb(196,80,78) , 29px 29px 0 rgb(196,80,78) , 30px 30px 0 rgb(196,80,78) , 31px 31px 0 rgb(196,80,78) , 32px 32px 0 rgb(196,80,78) , 33px 33px 0 rgb(196,80,78) , 34px 34px 0 rgb(196,80,78) , 35px 35px 0 rgb(196,80,78) , 36px 36px 0 rgb(196,80,78) , 37px 37px 0 rgb(196,80,78) , 38px 38px 0 rgb(196,80,78) , 39px 39px 0 rgb(196,80,78) , 40px 40px 0 rgb(196,80,78) , 41px 41px 0 rgb(196,80,78) , 42px 42px 0 rgb(196,80,78) , 43px 43px 0 rgb(196,80,78) , 44px 44px 0 rgb(196,80,78) , 45px 45px 0 rgb(196,80,78) , 46px 46px 0 rgb(196,80,78) , 47px 47px 0 rgb(196,80,78) , 48px 48px 0 rgb(196,80,78) , 1px 1px 0 rgba(196,80,78,0.980392) , 2px 2px 0 rgba(196,80,78,0.960784) , 3px 3px 0 rgba(196,80,78,0.941176) , 4px 4px 0 rgba(196,80,78,0.921569) , 5px 5px 0 rgba(196,80,78,0.901961) , 6px 6px 0 rgba(196,80,78,0.882353) , 7px 7px 0 rgba(196,80,78,0.862745) , 8px 8px 0 rgba(196,80,78,0.843137) , 9px 9px 0 rgba(196,80,78,0.819608) , 10px 10px 0 rgba(196,80,78,0.8) , 11px 11px 0 rgba(196,80,78,0.780392) , 12px 12px 0 rgba(196,80,78,0.760784) , 13px 13px 0 rgba(196,80,78,0.741176) , 14px 14px 0 rgba(196,80,78,0.721569) , 15px 15px 0 rgba(196,80,78,0.701961) , 16px 16px 0 rgba(196,80,78,0.682353) , 17px 17px 0 rgba(196,80,78,0.658824) , 18px 18px 0 rgba(196,80,78,0.639216) , 19px 19px 0 rgba(196,80,78,0.619608) , 20px 20px 0 rgba(196,80,78,0.6) , 21px 21px 0 rgba(196,80,78,0.580392) , 22px 22px 0 rgba(196,80,78,0.560784) , 23px 23px 0 rgba(196,80,78,0.541176) , 24px 24px 0 rgba(196,80,78,0.521569) , 25px 25px 0 rgba(196,80,78,0.498039) , 26px 26px 0 rgba(196,80,78,0.478431) , 27px 27px 0 rgba(196,80,78,0.458824) , 28px 28px 0 rgba(196,80,78,0.439216) , 29px 29px 0 rgba(196,80,78,0.419608) , 30px 30px 0 rgba(196,80,78,0.4) , 31px 31px 0 rgba(196,80,78,0.380392) , 32px 32px 0 rgba(196,80,78,0.360784) , 33px 33px 0 rgba(196,80,78,0.341176) , 34px 34px 0 rgba(196,80,78,0.317647) , 35px 35px 0 rgba(196,80,78,0.298039) , 36px 36px 0 rgba(196,80,78,0.278431) , 37px 37px 0 rgba(196,80,78,0.258824) , 38px 38px 0 rgba(196,80,78,0.239216) , 39px 39px 0 rgba(196,80,78,0.219608) ;

}
#controls button:hover {
  background: linear-gradient(180deg, #ce6161 0, #ef6664 100%);
}
#controls button:active {
  top: 5px;
  box-shadow: 0 5px 0 0 rgba(178,49,49,1) ;
}
#controls button:disabled {
  box-shadow: 0 10px 0 0 #999;
  background:linear-gradient(180deg, #c7c4c4 0, #b3b0b0 100%);
  text-shadow: none;
  cursor:not-allowed;
}

/*Animated Button*/
#game-menu ol{list-style: none;}
#game-menu ol li{margin: 20px 0px;text-align: center;}
.btn {
  background: #eb1f48;
  border-radius: 5px;
  color: white;
  cursor: pointer;

  font-size: 1.6em;

  letter-spacing: .2px;

  text-align: center;
  -webkit-user-select: none;
     -moz-user-select: none;
      -ms-user-select: none;
          user-select: none;
  width: 250px;
  border: none;
  outline: none;
  padding:12px;
}
.btn:hover {
  background: #ec3257;
}
.btn:active {
  box-shadow: inset 0px 2px 8px -1px #d7143b;
}
.btn:disabled {
  background: #a4a8ab;
  cursor: not-allowed;
}
a.btn{
  display: inline-block;
  box-sizing: border-box;
  text-decoration: none;
}

.next-level-button {
    position: absolute;
    top: 408px;
    right:32px;
    width: 100%;
    text-align: right;
}

.next-level-button button, .main-menu-btn.btn{
    width: auto;
    height:auto;
    line-height: normal;
    font-size: 16px;
    padding: 10px;
}
.main-menu-btn.btn{
  padding: 5px 15px;
  margin: -2px 13px 0px 80px;
  float:right;
}

.level-cleared{
  position: absolute;
  top: 53px;
  width: 100%;
}

.level-cleared p{
  font-size: 30px;
  clear: both;
  text-align: center;
  color: #fdfcfc;
}
.level-cleared button{
  font-size: 28px;
  width: auto;
  height:auto;
  line-height: normal;
  padding: 10px;
}
.level-cleared .highlight{
  color: #d03655;
  font-weight: 700;
}
.level-cleared img{width:35%;}

.error{
  font-size: 26px;
    color: red;
    width: 96%;
    text-align: center;
    border: 1px solid rgba(245, 245, 245, 0.51);
    background: rgba(255, 255, 255, 0.37);
    /* display: inline-block; */
    padding: 8px;
    position: absolute;
    top: 15px;
    z-index: 1;
    border-radius: 16px;
}
/*Vue.js transition*/
/* Enter and leave animations can use different */
/* durations and timing functions.              */
.slide-fade-enter-active {
  transition: all .5s ease;
}
.slide-fade-leave-active {
  transition: all .8s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.slide-fade-enter, .slide-fade-leave-to
/* .slide-fade-leave-active below version 2.1.8 */ {
  transform: translateX(10px);
  opacity: 0;
}

/*Disqus Comment*/
#disqus_thread {background: rgba(255, 255, 255, 0.24);padding: 10px;border-radius: 14px;}

body {
  background-color: #0FAFBB;
}

.aquarium {
  position: absolute;
  top: 315px;
  animation: swim 4s infinite linear, movement 50s infinite linear;
}

.aquarium1 {
  position: absolute;
  right:50%;
  animation: swimback 4s infinite linear, movement2 20s infinite linear;
}

.nemo {
  margin: 150px 50px 150px 100px;
  width: 80px;
  height: 65px;
  background-color: #F66702;
  border-radius: 50%;
  position: relative;
  box-shadow:  0px 0px 0px 1px #DF3A05,
    -3px -4px 0px 1px #ED5300 inset;
}

@keyframes swim {
  0% {transform: translate3d(0px,30px,0px);}
  20% {transform: translate3d(0px,0px,50px);}
  60% {transform: translate3d(30px,30px,0px);}
  100% {transform: translate3d(0px,30px,0px);}
}

@keyframes swimback {
  0% {transform: translate3d(30px,30px,30px);}
  20% {transform: translate3d(20px,20px,50px);}
  60% {transform: translate3d(10px,10px,50px);}
  100% {transform: translate3d(30px,30px,30px);}
}

@keyframes movement {
  0% {right:-100px;}
  100% {right:110%;}
}
@keyframes movement2 {
  0% {right:-100px;}
  100% {right:200%;}
}

.nemo-pouter {
  /*margin: -150px 300px;*/
}

.nemo:before {
  content: '';
  width: 1px;
  height: 3px;
  background-color: #0172F8;
  border-radius: 50%;
  position: absolute;
  top: 10px;
  left: 15px;
  transform: scaleX(0.7);
  box-shadow:
    4px 1px 1px 2px #000,
    3px -1px 1px 2px white,
    4px 0px 1px 6px #0172F8,
    4px 0px 0px 6px #000,
    1px -2px 0 11px #fff,
    1px -2px 2px 11px #000,
    52px 3px 1px 2px #000,
    51px 1px 1px 2px white,
    52px 2px 1px 7px #0172F8,
    52px 2px 0px 7px #000,
    50px 0px 0 12px #fff,
    50px 0px 2px 12px #000,
    15px 2px 0px 30px #F66702,
    14px 0px 0px 29px #c83a03,
    55px 2px 0px 30px #F66702,
    55px 0px 0px 30px #DF3A05,
    55px 0px 0px 30px #F58D58;
}

.n-collar {
  width: 0px;
  height: 0px;
  border-left: 8px solid transparent;
  border-bottom: 25px solid #F66702;
  border-right: 8px solid transparent;
  border-radius: 20px;
  position: absolute;
  bottom: -16px;
  left: 35px;
  z-index: -1;
  transform: rotate(30deg);
}

.n-collar:before {
  content: '';
  width: 0px;
  height: 0px;
  border-left: 8px solid transparent;
  border-bottom: 25px solid #F66702;
  border-right: 8px solid transparent;
  border-radius: 20px;
  position: absolute;
  bottom: -20px;
  left: 3px;
  z-index: -1;
  transform: rotate(-55deg);
}

.n-fin1 {
  width: 35px;
  height: 30px;
  background-color: #F66702;
  border-radius: 10px 10px 10px 100px;
  box-shadow: -2px 4px 1px 1px #fff,
    -3px 5px 1px 1px #000,
    -15px -8px 5px 0 #F35008 inset;
  position: absolute;
  top: 42px;
  left: -12px;
  z-index: -1;
  perspective: 1000px;
  perspective-origin: right;
  animation: fin1move 1s infinite;
}

@keyframes fin1move {
    0% {
	   transform: rotate3d(0, 0, 1, 20deg);
    }
    25% {
     transform: rotate3d(0, 2, 1, 20deg);
    }
    50% {
      transform: rotate3d(0, -2, 1, 20deg);
    }
    100% {
	   transform: rotate3d(0, 0, 1, 20deg);
    }
}

.n-smile {
  display: block;
  width: 13px;
  height: 12px;
  border-radius: 0 0 50% 50%;
  background-color: #7F0301;
  position: absolute;
  right: 40px;
  top: 40px;
}

.n-smile:before {
  content: '';
  height: 0;
  width: 0;
  border-left: 5px solid transparent;
  border-right: 5px solid transparent;
  border-bottom: 10px solid #7F0301;
  position: absolute;
  bottom: 5px;
  right: 10px;
  transform: rotate(-60deg);
}

.n-smile:after {
  content: '';
  height: 0;
  width: 0;
  border-left: 5px solid transparent;
  border-right: 5px solid transparent;
  border-bottom: 10px solid #7F0301;
  position: absolute;
  bottom: 5px;
  right: -7px;
  transform: rotate(60deg);
}

.n-tongue {
  width: 12px;
  height: 5px;
  background-color: #FE7878;
  border-radius: 50%;
  position: absolute;
  bottom: 0px;
  left: 1px;
  box-shadow: 0px -2px 0px 1px #D85456;
  z-index: 1;
}

.n-tongue:before {
  content: '';
  width: 8px;
  height: 7px;
  border-radius: 0 0 7px 7px;
  border-bottom: 1px solid;
  position: absolute;
  bottom: -7px;
  left: 2px;
  transform: scaleY(0.7);
  filter: blur(1px);
}

.n-grin {
  width: 40px;
  height: 20px;
  border-radius: 0 0 50px 50px;
  border-bottom: 3px solid #7F0301;
  transform: scaleY(0.5) rotate(10deg);
  position: absolute;
  top: -16px;
  left: -15px;
  filter: blur(1px);
}

.n-grin:before {
  content: '';
  display: block;
  width: 3px;
  height: 6px;
  border-radius: 3px 0 0 3px;
  border-left: 1px solid #7F0301;
  position: absolute;
  left: -1px;
  top: 8px;
  transform: scaleY(2) rotate(50deg);
}

.n-grin:after {
  content: '';
  display: block;
  width: 3px;
  height: 7px;
  border-radius: 3px 0 0 3px;
  border-right: 1px solid #7F0301;
  position: absolute;
  left: 39px;
  top: 2px;
  transform: scaleY(2) rotate(120deg);
}

.n-brows {
  width: 10px;
  height: 10px;
  border-radius: 15px 200px 0 0;
  position: absolute;
  top: -10px;
  left: 50px;
  border-top: 1px solid #7F0301;
  filter: blur(1px);
}

.n-brows:before {
  content: '';
  width: 10px;
  height: 10px;
  border-radius: 200px 24px 0 0;
  position: absolute;
  top: -2px;
  left: -38px;
  border-top: 1px solid #7F0301;
}

.n-brows:after {
  content: '';
  width: 25px;
  height: 25px;
  border-radius: 200px 24px 0 0;
  position: absolute;
  top: -10px;
  left: -45px;
  border-top: 1px solid #FD9D5A;
  filter: brightness(800%);
  transform: rotate(20deg);
}

.n-body {
  width: 95px;
  height: 90px;
  border-radius: 160px 180px 180px 150px;
  background-color: #F66702;
  position: absolute;
  top: -27px;
  left: 12px;
  z-index: -1;
  transform: rotate(-6deg);
}

.n-body:after {
  content: '';
  width: 80px;
  height: 80px;
  border-radius: 60px 60px 90px 0px;
  background-color: #F66702;
  position: absolute;
  top: 5px;
  right: -10px;
  transform: rotate(20deg);
}

.n-body:before {
  content: '';
  width: 25px;
  height: 30px;
  background-color: #F66702;
  border-radius: 50px 60px 60px 60px;
  box-shadow: -3px -1px 0 1px #fff,
    -3px -1px 2px 1px #000;
  transform: rotate(80deg);
  position: absolute;
  top: -16px;
  left: 35px;
  perspective: 1000px;
  perspective-origin: right;
  animation: tailafter-mov 1s infinite;
}

.n-fin {
  width: 30px;
  height: 30px;
  border-radius: 50px 400px 50px 50px;
  background-color: #F66702;
  position: absolute;
  top: 40px;
  right: -30px;
  transform: rotateZ(60deg);
  box-shadow: 3px -3px 1px 1px #fff,
    4px -3px 1px 1px #000,
    -1px -1px 0px -1px #000,
    -15px -1px 5px 0 #F35008 inset;
  z-index: 2;
  perspective: 1000px;
  perspective-origin: right;
  animation: fin-mov 1s infinite linear;
  transform-style: preserve-3d;
}

@keyframes fin-mov {
    0% {transform: rotate3d(0, 0, 1, 60deg);}
    25% {transform: rotate3d(0, -1, 1, 60deg);}
    100% {transform: rotate3d(0, 0, 1, 60deg);}
}

.n-tail:before {
  content: '';
  width: 30px;
  height: 30px;
  border-radius: 50px 400px 50px 50px;
  background-color: #F66702;
  position: absolute;
  top: -17px;
  right: -20px;
  transform: rotateZ(0deg);
  box-shadow: 3px -3px 1px 1px #fff,
    4px -4px 1px 1px #000,
    -15px -1px 2px 0 #F35008 inset;
  perspective: 1000px;
  animation: tail-mov 1s infinite linear;
  transform-style: preserve-3d;
}

@keyframes tail-mov {
    0% {transform: rotate3d(0, 0, 0, 30deg);}
    50% {transform: rotate3d(-1, -1, 0, 40deg);}
    100% {transform: rotate3d(0, 0, 0, 30deg);}
}

.n-tail {
  width: 50px;
  height: 50px;
  border-radius: 80px 0px 80px 100px;
  background-color: #F66702;
  position: absolute;
  top: -15px;
  right: -42px;
  transform: rotateZ(40deg);
  z-index: -1;
}

.n-tail:after {
  content: '';
  width: 25px;
  height: 40px;
  background-color: #F66702;
  border-radius: 50px 60px 60px 60px;
  box-shadow: -3px -1px 0 1px #fff,
    -3px -1px 2px 1px #000,
    -15px -1px 2px 0 #F35008 inset;
  transform: rotateZ(80deg);
  position: absolute;
  top: -23px;
  left: -10px;
  perspective: 1000px;
  perspective-origin: right;
  animation: tailafter-mov 1s infinite;
}

@keyframes tailafter-mov {
    0% {
	   transform: rotate3d(0, 0, 1, 80deg);
    }
    50% {
     transform: rotate3d(-0.2, -0.2, 1, 80deg);
    }
    100% {
	   transform: rotate3d(0, 0, 1, 80deg);
    }
}
.n-stripe {
  width: 8px;
  height: 20px;
  border-radius: 50%;
  background-color: #fff;
  position: absolute;
  right: -38px;
  top: 7px;
  box-shadow: -2px 1px 0 0,
    2px -1px 0 0,
    0px -13px 0 0,
    -1px -12px 0 1px;
}

.n-stripe:before {
  content: '';
  width: 8px;
  height: 25px;
  transform: rotate(-20deg);
  background-color: #fff;
  position: absolute;
  top: -18px;
  left: -3px;
  border-radius: 50%;
  box-shadow: -2px -3px 0 -1px,
    3px -1px 0 -1px;
}

.n-forestripe {
  width: 14px;
  height: 12px;
  background-color: #fff;
  position: absolute;
  right: -8px;
  top: -23px;
  z-index: 1;
  transform: rotate(12deg);
  border-radius: 0 50px 0px 50px;
  box-shadow: 3px -1px 0 -1px,
    -2px 1px 0 0px,
    7px 5px 0 0;
}

.n-forestripe:before {
  content: '';
  width: 8px;
  height: 50px;
  background: #fff;
  position: absolute;
  left: 16px;
  top: 0px;
  transform: rotate(-30deg);
  border-radius: 50%;
  z-index: 1;
}

.n-forestripe:after {
  content: '';
  width: 10px;
  height: 30px;
  position: absolute;
  right: -22px;
  top: 33px;
  background-color: #fff;
  transform: rotate(-15deg);
  border-radius: 50%;
  box-shadow: 3px 0px 0 -1px inset,
    -3px -3px 0 -1px inset,
    -5px -25px 0 0,
    -2px -18px 0 0,
    -1px -15px 0 0,
    -10px -35px 0 -3px;
}

.n-lips {
  width: 7px;
  height: 13px;
  background: #fc822a;
  border-radius: 26px 5px 5px 26px;
  transform: rotate(15deg);
  box-shadow: -1px -1px 0 0 #f35008,
    1px 1px 0 0 #f35008 inset,
    -1px -3px 0 -1px #f35008 inset;
  position: absolute;
  top: 30px;
  left: 24px;
  z-index: 2;
}

.n-lips:before {
  content: '';
  width: 18px;
  height: 10px;
  background: #fc822a;
  border-radius: 0 0 16px 16px;
  position: absolute;
  left: 3px;
  top: 2px;
  transform: rotate(25deg);
  box-shadow: -1px 0px 0 1px #f35008 inset,
    -3px -2px 0 1px #f35008 inset;

}

.n-lips:after {
  content: '';
  width: 20px;
  height: 10px;
  background: #fc822a;
  border-radius: 0 0 16px 16px;
  position: absolute;
  left: 2px;
  top: 10px;
  transform: rotate(-15deg);
  box-shadow: 0px -1px 0 1px #f35008 inset,
    -1px -1px 0 2px #f35008 inset;
}

.n-mouth {
  content: '';
  width: 5px;
  height: 4px;
  border-radius: 10px 10px 10px 2px;
  background: #7F0A02;
  position: absolute;
  top: 8px;
  left: 5px;
  z-index: 3;
  transform: rotate(110deg);
}


.bubble {
  z-index: 3;
  width: 10px;
  height: 8px;
  margin: 10px;
  border-radius: 50%;
  box-shadow: 0px 0px 2px 0px,
    -1px 2px 0px -1px #fff inset;
  position: absolute;
  transform: translate3d(15px, 30px , 0px);
  animation: shoot-bubble 2s infinite linear;
}

.bubble:before {
  content: '';
   width: 10px;
  height: 10px;
  border-radius: 50%;
  box-shadow: 0px 0px 2px 0px,
    -1px 2px 0 -1px #fff inset;
  position: absolute;
  transform: translate3d(-25px, -30px , 0px);
  animation: shoot-bubble-before 2s infinite linear;
}

.bubble:after {
  content: '';
  width: 10px;
  height: 10px;
  border-radius: 50%;
  box-shadow: 0px 0px 2px 0px,
    -1px 2px 0 -1px #fff inset;
  position: absolute;
  transform: translate3d(-25px, -60px , 0px);
  animation: shoot-bubble-after 2s infinite linear;
}


@keyframes shoot-bubble {

  0% {transform: translate3d(15px, 30px , 0px);}
  50% {transform: translate3d(-25px, -30px , 0px);}
  70% {transform: translate3d(-35px, -100px , 0px);}
  100% {transform: translate3d(-35px, -180px , 0px);}
}

 @keyframes shoot-bubble-before {
  0% {transform: translate3d(-25px, -30px , 0px);}
  50% {transform: translate3d(-25px, -100px , 0px);}
  100% {transform: translate3d(-25px, -180px , 0px);}
}

@keyframes shoot-bubble-after {
  0% {transform: translate3d(-25px, -100px , 0px);}
  50% {transform: translate3d(-25px, -180px , 0px);}
  100% {transform: translate3d(-25px, -250px , 0px);}
}

</style>
