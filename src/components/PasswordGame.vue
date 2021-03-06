<script>
  import { PASSWORD_STATUS } from "../constants";
  import { reactive, toRefs, computed, watch, onMounted } from "vue";

  export default {
    setup(props, { emit }) {
      const state = reactive({
        status: "In Progress",
        passwordInput: "",
        correctPassword: 0,
        gameStatus: computed(() => {
          let styles = "";
          let text = "Locked";
          if (state.status === PASSWORD_STATUS.FAIL) {
            styles = "is-red";
            text = "Access Denied";
          } else if (state.status === PASSWORD_STATUS.PASS) {
            styles = "is-green";
            text = "Access Granted";
          }

          return { styles, text };
        }),
        userWins: computed(() => state.status === PASSWORD_STATUS.PASS),
      });

      const methods = {
        checkPassword: () => {
          if (state.correctPassword === state.passwordInput) {
            state.status = PASSWORD_STATUS.PASS;
            emit("mini-game-won", "password-game");
          } else {
            state.status = PASSWORD_STATUS.FAIL;
          }
        },
        generateNewPassword: () => {
          return Math.floor(Math.random() * 1000000 + 1000).toString();
        },
      };

      onMounted(() => {
        state.correctPassword = methods.generateNewPassword();
      });

      watch(
        () => state.gameState,
        currentGameState => {
          if (currentGameState === PASSWORD_STATUS.FAIL) {
            state.correctPassword = methods.generateNewPassword();
          }
        }
      );

      return {
        ...toRefs(state),
        ...methods,
      };
    },
  };
</script>

<template>
  <form class="password-task" @submit.prevent>
    <p class="password-task-status" :class="gameStatus.styles">
      <span>{{ gameStatus.text }}</span>
    </p>
    <div class="password-task-input">
      <label for="enter-password">Enter Password</label>
      <input
        id="enter-password"
        class="nes-input"
        type="text"
        v-model="passwordInput"
        @keyup.enter="checkPassword"
      />
    </div>
    <p class="post-it nes-container">
      Secret Password: <br />{{ correctPassword }}
    </p>
    <button
      class="password-task-submit nes-btn is-primary"
      type="submit"
      @click="checkPassword"
    >
      Submit
    </button>
  </form>
</template>

<style>
  .password-task {
    display: grid;
  }

  .password-task-input {
    display: grid;
    margin-bottom: 30px;
  }

  .password-task-input label {
    margin-bottom: 1rem;
    font-size: 1.5rem;
  }

  .password-task-input input {
    padding: 0.5rem;
    font-size: 1.5rem;
    text-align: center;
  }

  .password-task-submit {
    font-size: 1.2rem;
  }

  .password-task-status {
    padding: 1rem;
    padding-top: 1.25rem;
    margin-top: 0.5rem;
    margin-bottom: 1.5rem;
    color: #ccc;
    border: 5px dashed #ccc;
  }

  .password-task-status.is-green {
    border-color: var(--green);
  }

  .password-task-status.is-red {
    border-color: var(--red);
  }

  .post-it {
    position: absolute;
    bottom: -200px;
    right: -200px;
    padding: 1rem;
    width: 300px;
    background-color: yellow;
    color: black;
  }
</style>
