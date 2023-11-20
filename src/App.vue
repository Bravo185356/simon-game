<template>
  <div id="app">
    <div class="wrapper">
      <div class="buttons">
        <button
          :class="{ active: glowButton == 1, hoverable: !buttonsBlock }"
          @click="checkButton(1)"
          class="button blue"
          :disabled="buttonsBlock"
        ></button>
        <button
          :class="{ active: glowButton == 2, hoverable: !buttonsBlock }"
          @click="checkButton(2)"
          class="button red"
          :disabled="buttonsBlock"
        ></button>
        <button
          :class="{ active: glowButton == 3, hoverable: !buttonsBlock }"
          @click="checkButton(3)"
          class="button yellow"
          :disabled="buttonsBlock"
        ></button>
        <button
          :class="{ active: glowButton == 4, hoverable: !buttonsBlock }"
          @click="checkButton(4)"
          class="button green"
          :disabled="buttonsBlock"
        ></button>
      </div>
      <div>
        <div class="lose-message" v-if="showLoseMessage">Вы ошиблись</div>
        <div v-for="difficult in difficulties">
          <input
            @click="selectDifficult = difficult"
            type="radio"
            :id="'difficult' + difficult.time"
            name="difficult"
            :checked="selectDifficult.name === difficult.name"
          />
          <label :for="'difficult' + difficult.time">{{ difficult.name }} {{ difficult.time }}</label>
        </div>
        <div class="level">Уровень: {{ level }}</div>
        <button class="start-button" :disable="startButtonDisable" @click="generateSequence">Старт</button>
      </div>
    </div>
  </div>
</template>
<script>
import Sound1 from "./assets/audio/sounds_1.mp3";
import Sound2 from "./assets/audio/sounds_2.mp3";
import Sound3 from "./assets/audio/sounds_3.mp3";
import Sound4 from "./assets/audio/sounds_4.mp3";
export default {
  data() {
    return {
      sequence: [],
      pushedButtons: [],
      level: 1,
      startButtonDisable: false,
      glowButton: "",
      buttonsBlock: true,
      showLoseMessage: false,
      difficulties: [
        { name: "Легко", time: 1500 },
        { name: "Средне", time: 1000 },
        { name: "Сложно", time: 400 },
      ],
      selectDifficult: { name: "Легко", time: 1500 },
    };
  },
  methods: {
    async generateSequence() {
      if (this.sequence.length < this.level) {
        const randomNumber = Math.floor(Math.random() * 4 + 1);
        this.sequence.push(randomNumber);
        this.glowButton = randomNumber;

        const audio = new Audio(this.getSoundFile(randomNumber));
        audio.volume = 1;
        audio.play();

        await this.executeFunctionWithTimeout(() => (this.glowButton = ""), this.selectDifficult.time);
        await this.executeFunctionWithTimeout(() => this.generateSequence(), 200);
      }
      if (this.sequence.length == this.level && this.buttonsBlock) {
        this.buttonsBlock = false;
      }
      this.startButtonDisable = true;
    },
    getSoundFile(number) {
      switch (number) {
        case 1:
          return Sound1;
          break;
        case 2:
          return Sound2;
          break;
        case 3:
          return Sound3;
          break;
        case 4:
          return Sound4;
          break;
      }
    },
    checkButton(button) {
      if (!this.sequence.length) {
        return console.log("Нажмите кнопку старт");
      }

      const audio = new Audio(this.getSoundFile(button));
      audio.volume = 1;
      audio.play();

      if (this.sequence[this.pushedButtons.length] === button) {
        this.pushedButtons.push(button);
      } else {
        this.changeLevel(1);
        this.startButtonDisable = false;
        this.buttonsBlock = true;
        this.showLoseMessage = true;
      }
      if (this.pushedButtons.length === this.level) {
        this.buttonsBlock = true;
        this.changeLevel(this.level + 1);
        setTimeout(() => {
          this.generateSequence();
        }, 500);
      }
    },
    changeLevel(level) {
      this.level = level;
      this.pushedButtons = [];
      this.sequence = [];
    },
    async executeFunctionWithTimeout(callback, timeout) {
      await new Promise((resolve, reject) => {
        setTimeout(async () => {
          await callback();
          resolve();
        }, timeout);
      });
    },
  },
};
</script>

<style scoped>
.wrapper {
  display: flex;
  align-items: center;
  gap: 100px;
}
.buttons {
  display: grid;
  justify-content: start;
  grid-template: repeat(2, auto) / repeat(2, auto);
  width: 600px;
  height: 600px;
  background-color: rgb(17, 17, 17);
  border-radius: 50%;
  overflow: hidden;
}
button {
  border: 0;
}
.blue.active,
.blue.hoverable {
  background-color: rgb(98, 98, 255);
}
.red.active,
.red.hoverable {
  background-color: rgb(252, 78, 78);
}
.yellow.active,
.yellow.hoverable {
  background-color: rgb(255, 255, 70);
}
.green.active,
.green.hoverable {
  background-color: rgb(62, 252, 62);
}
.button {
  width: 300px;
  height: 300px;
}
.blue {
  background-color: rgb(169, 169, 255);
  border-radius: 50% 0 0 0;
}
.red {
  background-color: rgb(255, 158, 158);
  border-radius: 0 50% 0 0;
}
.yellow {
  background-color: rgb(255, 255, 161);
  border-radius: 0 0 0 50%;
}
.green {
  background-color: rgb(126, 255, 126);
  border-radius: 0 0 50% 0;
}
.start-button {
  width: 200px;
  height: 40px;
  background-color: rgb(82, 82, 255);
  border-radius: 10px;
  color: white;
  font-size: 20px;
  margin-top: 20px;
}
.level {
  font-size: 26px;
  text-align: center;
}
.lose-message {
  font-size: 32px;
}
</style>
