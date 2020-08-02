<template>
    <div id="app">
        <div class="container">
            <h1>Simon Says</h1>
            <div class="simon">
                <ul @click="checkUserChoice">
                    <li class="red" data-tile="1"></li>
                    <li class="blue" data-tile="2"></li>
                    <li class="yellow" data-tile="3"></li>
                    <li class="green" data-tile="4"></li>
                </ul>
            </div>
            <div class="game-info">
                <h2>Round: <span data-round="0"> {{this.round}}</span></h2>
                <button data-action="start" v-on:click="startGame">Start</button>
                <p data-action="lose" id="lostParagraph"></p>
            </div>
            <div class="game-options">
                <h2>Game Options:</h2>
                <input type="radio" name="mode" value="Easy" checked>Easy<br>
                <input type="radio" name="mode" value="Normal">Normal<br>
                <input type="radio" name="mode" value="Hard">Hard<br>
            </div>
            <div data-action="sound" id="soundContainer">

            </div>
        </div>
    </div>
</template>

<script>

  export default {
    name: "App",
    data: function () {
      return {
        sequence: [],
        config: {
          easy: {
            activeTime: 1500,
            delayTime: 500,
          },
          normal: {
            activeTime: 1000,
            delayTime: 200
          },
          hard: {
            activeTime: 400,
            delayTime: 100,
          }
        },
        round: 0,
        mode: '',
      }

    },

    components: {},
    computed: {},
    methods: {
      choseGameOption: function () {
        let modesInputs = document.getElementsByTagName('input');
        for (let input of modesInputs) {
          if (input.checked === true) {
            this.mode = `${input.value}`;
          }
        }
      },

      startGame: function () {
        let p = document.getElementById('lostParagraph');
        p.style.display = 'none';
        this.sequence = [];
        this.round = 1;
        this.choseGameOption();
        this.gameProcess();
      },

      loseGame: function () {
        let p = document.getElementById('lostParagraph');
        p.innerHTML = `Sorry, you lost after ${this.round} rounds!`;
        p.style.display = 'block';
        this.round = 0;
        let buttons = document.getElementsByTagName('li');
        for (let button of buttons) {
          button.classList.remove('hoverable');
        }
      },

      successGame: async function () {
        await this.sleep(1000);
        this.round++;
        this.gameProcess();
      },

      soundOfButton: function (trackNum) {
        let soundDiv = document.getElementById('soundContainer');
        let audio = document.createElement('audio');
        audio.setAttribute('autoplay', '');
        audio.innerHTML = `<source src="sounds/${trackNum}.ogg"/>`
        soundDiv.append(audio);
      },

      checkUserChoice: async function (e) {
        if (this.round === 0) return
        if (this.sequence.length === 0) return;
        if (e.target.tagName !== 'LI') return;
        this.soundOfButton(e.target.dataset.tile);
        e.target.classList.toggle('active');
        await this.sleep(400);
        e.target.classList.toggle('active');
        let expectedButton = this.sequence.shift();
        if (expectedButton !== parseInt(e.target.dataset.tile)) {
          this.loseGame();
          return
        }
        if (this.sequence.length === 0) {
          await this.successGame();
        }
      },

      gameProcess: function () {
        let buttons = document.getElementsByTagName('li');
        for (let button of buttons) {
          button.classList.add('hoverable');
        }
        this.randomButtons();
        this.performButtons()
      },

      sleep: async function (delay) {
        return new Promise((resolve) => {
          setTimeout(() => {
            resolve()
          }, delay);
        })
      },

      performButtons: async function () {
        for (let number of this.sequence) {
          let li = this.getLiByDataset(number);
          if (this.mode === 'Easy') {
            await this.sleep(this.config.easy.delayTime);
          } else if (this.mode === 'Normal') {
            await this.sleep(this.config.normal.delayTime);
          } else {
            await this.sleep(this.config.hard.delayTime);
          }

          li.classList.add('active');
          this.soundOfButton(number);

          if (this.mode === 'Easy') {
            await this.sleep(this.config.easy.activeTime);
          } else if (this.mode === 'Normal') {
            await this.sleep(this.config.normal.activeTime);
          } else {
            await this.sleep(this.config.hard.activeTime);
          }
          li.classList.remove('active');
        }
      },

      getLiByDataset: function (id) {
        let list = document.getElementsByTagName('li');
        for (let li of list) {
          if (id === parseInt(li.dataset.tile)) {
            return li
          }
        }
      },

      randomButtons: function () {
        for (let i = 0; i < this.round; i++) {
          this.randomNumber(1, 4);
        }
      },

      randomNumber: function (min, max) {
        let rand = min - 0.5 + Math.random() * (max - min + 1);
        let finalNumber = Math.round(rand);
        this.sequence.push(finalNumber);
      }
    }

  }

</script>

<style lang="scss">

    body {
        font-family: Arial, serif;
        color: #333;
        -webkit-user-select: none; /* Chrome/Safari */
        -moz-user-select: none; /* Firefox */
        -ms-user-select: none; /* IE10+ */
        -o-user-select: none;
        user-select: none;
    }

    #app {
        h1 {
            margin: 1em 0 2em;
            text-align: center;
        }

        ul {
            list-style: none;
        }

        ul, li {
            padding: 0;
            margin: 0;
        }

        p[data-action="lose"] {
            display: none;
        }

        .active {
            animation-timing-function: linear;
            opacity: 1 !important;
        }

        .clearfix {
            width: 100%;
            clear: both;
        }

        .container {
            width: 540px;
            margin: 0 auto;
        }

        .simon {
            background: #fff;
            position: relative;
            float: left;
            margin-right: 3em;
            width: 302px;
            height: 295px;
            -webkit-border-radius: 150px 150px 150px 150px;
            border-radius: 150px 150px 150px 150px;
            -moz-box-shadow: 2px 1px 12px #aaa;
            -webkit-box-shadow: 2px 1px 12px #aaa;
            -o-box-shadow: 2px 1px 12px #aaa;
            box-shadow: 2px 1px 12px #aaa;
        }

        .red, .blue, .yellow, .green {
            opacity: 0.6;
            height: 290px;
            -webkit-border-radius: 150px 150px 150px 150px;
            border-radius: 150px 150px 150px 150px;
            position: absolute;
            text-indent: 10000px;
        }

        .red:hover, .blue:hover, .yellow:hover, .green:hover {
            border: 2px solid black
        }

        .red {
            background: #FF5643;
            clip: rect(0px, 300px, 150px, 150px);
            width: 296px;
        }

        .blue {
            background: dodgerblue;
            clip: rect(0px, 150px, 150px, 0px);
            width: 300px;
        }

        .yellow {
            background: #FEEF33;
            clip: rect(150px, 150px, 300px, 0px);
            width: 300px;
        }

        .green {
            background: #BEDE15;
            clip: rect(150px, 300px, 300px, 150px);
            width: 296px;
        }

        .game-info {
            margin-top: 90px;
        }

        .game-info button {
            width: 5em;
            box-sizing: border-box;
            font-size: 1.4em;
            -webkit-border-radius: 10px 10px 10px 10px;
            border-radius: 10px 10px 10px 10px;
            background: #6DABE8;
            border: none;
            padding: 0.3em 0.6em;
        }

        .game-info button:hover {
            background: #78BCFF;
        }

        .game-options h2 {
            margin-top: 30px;
            margin-bottom: 0;
        }

        .game-options input[type="radio"] {
            margin-right: 10px;
        }

        .hoverable:hover {
            cursor: pointer;
        }

    }


</style>
