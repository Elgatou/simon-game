<template>
  <div class="game">
    <div class="pad">
      <button
        @click="userInputHandler('blue')"
        class="blue"
        :class="{ active: activeButton === 'blue' }"
      ></button>
      <button
        @click="userInputHandler('yellow')"
        class="yellow"
        :class="{ active: activeButton === 'yellow' }"
      ></button>
      <button
        @click="userInputHandler('green')"
        class="green"
        :class="{ active: activeButton === 'green' }"
      ></button>
      <button
        @click="userInputHandler('red')"
        class="red"
        :class="{ active: activeButton === 'red' }"
      ></button>
    </div>
    <div class="info">
      <div class="settings">
        <p>уровень сложности:</p>
        <input type="radio" id="easy" value="easy" v-model="difficulty" />
        <label for="easy">Легкий</label>
        <input type="radio" id="normal" value="normal" v-model="difficulty" />
        <label for="normal">Нормальный</label>
        <input type="radio" id="hard" value="hard" v-model="difficulty" />
        <label for="hard">Сложный</label>
      </div>
      <button class="start" @click="init">Start</button>
      <p>score: {{ score }}</p>
      <p v-if="dontInterruptMessage">
        Если вы хотите перезапустить игру, пожалуйста дождитесь окончания
        автовоспроизведения последовательности и нажмите кнопку Start
      </p>
      <p v-if="lose">Вы проиграли, чтобы начать заного нажмите кнопку Start</p>
    </div>
  </div>
</template>

<script>
import redSound from "./1.mp3";
import blueSound from "./2.mp3";
import greenSound from "./3.mp3";
import yellowSound from "./4.mp3";

const sounds = {
  red: redSound,
  blue: blueSound,
  green: greenSound,
  yellow: yellowSound
};
export default {
  name: "Simon",
  data: function() {
    return {
      /**
       * state of the game
       *
       * @type {('' | 'USERAWAIT' | 'PLAYINGSEQUENCE' | 'FAIL' )}
       */
      state: "",
      activeButton: "",
      sequence: [],
      userSequence: [],
      difficulty: "normal",
      score: 0,
      dontInterruptMessage: false
    };
  },
  computed: {
    lose() {
      if (this.state === "FAIL") return true;
      return false;
    },
    delay() {
      const difficultyDiсt = {
        easy: 1500,
        normal: 1000,
        hard: 400
      };
      return difficultyDiсt[this.difficulty];
    }
  },
  methods: {
    init() {
      if (this.state === "PLAYINGSEQUENCE") {
        this.dontInterruptMessage = true;
        setTimeout(() => (this.dontInterruptMessage = false), 6000);
      } else {
        this.score = 0;
        this.sequence = [];
        this.userSequence = [];
        this.dontInterruptMessage = false;
        this.newRound();
      }
    },
    async newRound() {
      const getRandomColor = () => {
        const colors = Object.keys(sounds);
        return colors[Math.floor(Math.random() * colors.length)];
      };
      this.userSequence = [];
      this.state = "PLAYINGSEQUENCE";
      this.sequence.push(getRandomColor());
      await this.playSequence();
      this.state = "USERAWAIT";
    },

    async playSequence() {
      for (const color of this.sequence) {
        await this.delayActivateButton(color);
      }
    },

    delayActivateButton(color) {
      return new Promise(resolve => {
        setTimeout(() => {
          this.activateButton(color);
          resolve();
        }, this.delay);
      });
    },

    activateButton(color) {
      this.activeButton = color;
      setTimeout(() => (this.activeButton = ""), 300);
      const audio = new Audio(sounds[color]);
      audio.play();
    },

    userInputHandler(color) {
      if (this.state === "USERAWAIT") {
        this.activateButton(color);
        this.userSequence.push(color);

        for (let i = 0; i < this.userSequence.length; i++) {
          if (this.userSequence[i] !== this.sequence[i]) {
            this.state = "FAIL";
            return;
          }
        }

        if (this.userSequence.length === this.sequence.length) {
          this.score++;
          this.newRound();
        }
      }
    }
  }
};
</script>

<style scoped lang="scss">
.game {
  width: 800px;
  margin: 0 auto;
  display: flex;
}
.info {
  flex: 1;
}
.pad {
  flex: none;
  width: 400px;
  height: 400px;
  border-radius: 200px;
  overflow: hidden;
  display: flex;
  flex-wrap: wrap;
  margin: 0 auto;
}

button {
  width: 200px;
  border: none;

  &:hover {
    cursor: pointer;
  }

  &.start {
    padding: 10px;
    margin: 10px;
    font-size: 18px;
  }

  &.red {
    background-color: rgb(255, 86, 86);
    &.active {
      background-color: red;
    }
  }
  &.blue {
    background-color: rgb(90, 90, 243);
    &.active {
      background-color: blue;
    }
  }
  &.green {
    background-color: rgb(51, 206, 51);
    &.active {
      background-color: green;
    }
  }
  &.yellow {
    background-color: rgb(253, 253, 171);

    &.active {
      background-color: yellow;
    }
  }
}
</style>
