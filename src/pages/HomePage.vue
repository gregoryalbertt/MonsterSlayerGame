<template>
  <header>
    <h1>Monster Slayer</h1>
  </header>
  <div id="game">
    <div class="container">
      <div class="row" style="display: flex">
        <div class="col">
          <section id="player" class="container">
            <h2>Warrior</h2>
            <img class="spirit" :src="playerImage" alt="warrior" />
            <div class="healthbar">
              <div class="healthbar__value" :style="playerBarStyle"></div>
            </div>
          </section>
        </div>
        <div class="col">
          <section id="monster" class="container">
            <h2>Monster</h2>
            <img class="spirit" :src="monsterImage" alt="monster" />
            <div class="healthbar">
              <div class="healthbar__value" :style="monsterBarStyle"></div>
            </div>
          </section>
        </div>
      </div>
    </div>
    <section v-if="winner" class="container" id="gamemessage">
      <h2>Game Over!</h2>
      <h3 v-if="winner === 'player'">You won!</h3>
      <h3 v-else-if="winner === 'monster'">You Lost!</h3>
      <h3 v-else>Its a draw!</h3>
      <button @click="startNewGame">Start New Game</button>
    </section>
    <section v-else id="controls">
      <button :disabled="!endTurn" @click="attackMonster">ATTACK</button>
      <button @click="SpecialAttack" :disabled="mayUseSpecialAttack">
        SPECIAL ATTACK
      </button>
      <button @click="healPlayer" :disabled="!mayHeal">
        HEAL ({{ heals }})
      </button>
      <button :disabled="!endTurn" @click="surrender">SURRENDER</button>
    </section>
    <section id="log" class="container">
      <h2>Battle Log</h2>
      <ul>
        <li v-for="logMessage in logMessages" :key="logMessage">
          <span
            :class="{
              'log--player': logMessage.owner === 'player',
              'log--monster': logMessage.owner === 'monster',
            }"
          >
            {{ logMessage.owner }}
          </span>

          <span v-if="logMessage.action === 'attack'">
            attacks and deals
            <span class="log--damage">{{ logMessage.value }}</span>
          </span>
          <span v-else>
            heals by <span class="log--heal">{{ logMessage.value }}</span>
          </span>
        </li>
      </ul>
    </section>
  </div>
</template>

<script>
function getRandomValue(min, max) {
  return Math.floor(Math.random() * (max - min)) + min;
}

export default {
  data() {
    return {
      playerHealth: 100,
      monsterHealth: 100,
      currentRoud: 0,
      heals: 2,
      winner: "",
      logMessages: [],
      playerState: "idle",
      monsterState: "idle",
      endTurn: true,
    };
  },
  methods: {
    startNewGame() {
      this.playerHealth = 100;
      this.monsterHealth = 100;
      this.currentRoud = 0;
      this.heals = 2;
      this.winner = null;
      this.logMessages = [];
      this.playerState = "idle";
      this.monsterState = "idle";
    },
    attackMonster() {
      this.endTurn = false;
      this.playerState = "attacking";
      this.monsterState = "hit";
      setTimeout(() => {
        this.playerState = "idle";
        this.monsterState = "idle";
        const attackValue = getRandomValue(5, 12);
        this.monsterHealth -= attackValue;
        this.addLogMessage("player", "attack", attackValue);
        this.attackPlayer();
        this.currentRoud++;
      }, 580);
    },
    attackPlayer() {
      this.monsterState = "attacking";
      const attackValue = getRandomValue(8, 15);
      setTimeout(() => {
        this.monsterState = "idle";
        this.playerHealth -= attackValue;
        this.endTurn = true;
      }, 1200);
      this.addLogMessage("monster", "attack", attackValue);
    },
    SpecialAttack() {
      this.endTurn = false;
      this.playerState = "attacking";
      this.monsterState = "hit";
      const attackValue = getRandomValue(10, 25);
      setTimeout(() => {
        this.playerState = "idle";
        this.monsterState = "idle";
        this.monsterHealth -= attackValue;
        this.addLogMessage("player", "attack", attackValue);
        this.currentRoud++;
        this.attackPlayer();
      }, 1400);
    },
    healPlayer() {
      const healValue = getRandomValue(10, 25);
      if (healValue + this.playerHealth > 100) {
        this.playerHealth = 100;
      } else {
        this.playerHealth += healValue;
      }
      this.heals--;
      this.currentRoud++;
      this.addLogMessage("player", "heals", healValue);
      this.attackPlayer();
    },
    surrender() {
      this.winner = "monster";
      this.playerState = "dead";
    },
    addLogMessage(who, what, value) {
      this.logMessages.unshift({
        owner: who,
        action: what,
        value: value,
      });
    },
  },
  computed: {
    monsterBarStyle() {
      if (this.monsterHealth < 0) {
        return { width: "0px" };
      }
      return { width: this.monsterHealth + "%" };
    },
    playerBarStyle() {
      if (this.playerHealth < 0) {
        return { width: "0px" };
      }
      return { width: this.playerHealth + "%" };
    },
    mayUseSpecialAttack() {
      return this.endTurn ? this.currentRoud % 3 !== 0 : true;
    },
    mayHeal() {
      return this.endTurn ? this.heals : false;
    },
    playerImage() {
      if (this.playerState === "attacking") {
        return "warrior-attack.gif";
      } else if (this.playerState === "hit") {
        return "warrior-hit-and-dead.gif";
      } else if (this.playerState === "dead") {
        return "warrior-hit-and-dead.gif";
      }
      return "warrior-idle.gif";
    },
    monsterImage() {
      if (this.monsterState === "attacking") {
        return "monster-attack.gif";
      } else if (this.monsterState === "hit") {
        return "monster-hit.gif";
      } else if (this.monsterState === "dead") {
        return "monster-dead.gif";
      }
      return "monster-idle.gif";
    },
    gameOver() {
      return this.playerHealth <= 0;
    },
  },
  watch: {
    // playerHealth(data) {
    //   if (data <= 0 && this.monsterHealth <= 0) {
    //     this.winner = "draw";
    //   } else if (data < 0) {
    //     this.winner = "monster";
    //     this.playerState = "dead";
    //   }
    // },
    // monsterHealth(data) {
    //   if (data <= 0 && this.playerHealth <= 0) {
    //     this.winner = "draw";
    //   } else if (data < 0) {
    //     this.winner = "player";
    //     this.monsterState = "dead";
    //   }
    // },
    endTurn(endTurn) {
      if (endTurn) {
        if (this.playerHealth <= 0 && this.monsterHealth <= 0) {
          this.winner = "draw";
        } else if (this.playerHealth <= 0) {
          this.winner = "monster";
          this.playerState = "dead";
        } else if (this.monsterHealth <= 0) {
          this.winner = "player";
          this.monsterState = "dead";
        }
      }
    },
  },
};
</script>

<style>
* {
  box-sizing: border-box;
}

html {
  font-family: "Jost", sans-serif;
}

body {
  margin: 0;
}

header {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
  padding: 0.5rem;
  background-color: #880017;
  color: white;
  text-align: center;
  margin-bottom: 2rem;
}

section {
  width: 90%;
  max-width: 40rem;
  margin: auto;
}

.healthbar {
  width: 100%;
  height: 40px;
  border: 1px solid #575757;
  margin: 1rem 0;
  background: #fde5e5;
}

.healthbar__value {
  background-color: #00a876;
  width: 100%;
  height: 100%;
}

.container {
  text-align: center;
  padding: 0.5rem;
  margin: 1rem auto;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
  border-radius: 12px;
}

#monster h2,
#player h2 {
  margin: 0.25rem;
}

#controls {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
}

button {
  font: inherit;
  border: 1px solid #88005b;
  background-color: #88005b;
  color: white;
  padding: 1rem 2rem;
  border-radius: 12px;
  margin: 1rem;
  width: 12rem;
  cursor: pointer;
  box-shadow: 1px 1px 4px rgba(0, 0, 0, 0.26);
}

button:focus {
  outline: none;
}

button:hover,
button:active {
  background-color: #af0a78;
  border-color: #af0a78;
  box-shadow: 1px 1px 8px rgba(0, 0, 0, 0.26);
}

button:disabled {
  background-color: #ccc;
  border-color: #ccc;
  box-shadow: none;
  color: #3f3f3f;
  cursor: not-allowed;
}

#log ul {
  list-style: none;
  margin: 0;
  padding: 0;
}

#log li {
  margin: 0.5rem 0;
}

.log--player {
  color: #7700ff;
}

.log--monster {
  color: #da8d00;
}

.log--damage {
  color: red;
}

.log--heal {
  color: green;
}

.spirit {
  width: 100%;
}
</style>
