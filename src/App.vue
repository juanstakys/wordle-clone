<script lang="ts" setup>
import { onMounted, reactive, computed } from 'vue';
import SimpleKeyboard from './components/SimpleKeyboard.vue';
import WordRow from './components/WordRow.vue';

const state = reactive({
  solution: "frame",
  guesses: ["", "", "", "", "", ""],
  currentGuessIndex: 0,
  guessedLetters: {
    miss: [] as string[],
    found: [] as string[],
    hint: [] as string[],
  }
});

const wonGame = computed(
  () => state.guesses[state.currentGuessIndex - 1] == state.solution
);

const lostGame = computed(() => !wonGame.value
 && state.currentGuessIndex >= 6 
);

const handleInput = (key: string) => {
  if(state.currentGuessIndex >= 6 || wonGame.value) return;
  const currentGuess = state.guesses[state.currentGuessIndex];
  if (key == "{enter}") {
    // SEND GUESS
    if (currentGuess.length == 5) {
      state.currentGuessIndex++;
      for (var i = 0; i < currentGuess.length; i++) {
        let c = currentGuess.charAt(i);
        if(c == state.solution.charAt(i)) {
          state.guessedLetters.found.push(c);
        } else if (state.solution.includes(c)) {
          state.guessedLetters.hint.push(c);
        } else {
          state.guessedLetters.miss.push(c);
        }
      }
    }
  } else if (key == "{bksp}") {
    // REMOVE LAST LETTER
    state.guesses[state.currentGuessIndex] = currentGuess.slice(0, -1);
  } else if (key.length == 1) {
    // ADD LETTER IF ALPHABETICAL
    const alphaRegex = /^[a-zA-Z]+$/;
    if (alphaRegex.test(key)) {
      state.guesses[state.currentGuessIndex] += key;
    }
  }
};

onMounted(() => {
  window.addEventListener("keyup", (e) => {
    e.preventDefault();
    let key =
      e.key == 'Enter'
        ? "{enter}"
        : e.key == "Backspace"
        ? "{bksp}"
        : e.key.toLowerCase();
    handleInput(key);
  });
});

</script>

<template>
  <div class="flex flex-col h-screen max-w-md mx-auto justify-evenly">
    <div>
      <word-row 
        v-for="(guess, i) in state.guesses"
        :key="i"
        :value="guess"
        :solution="state.solution"
        :submitted="i < state.currentGuessIndex"
      />
    </div>
    <p v-if="wonGame" class="text-center">
      🏆 Congrats! You won!
    </p>
    <p v-else-if="lostGame" class="text-center">
      😢 Sorry, you lost. The solution was "{{ state.solution.toUpperCase() }}".
    </p>
    <simple-keyboard
      @onKeyPress="handleInput"
      :guessedLetters="state.guessedLetters"
      />
  </div>
</template>

<style></style>