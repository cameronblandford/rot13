<script lang="ts">
  import { onMount } from "svelte";

  let sentence = $state("");
  let encodedSentence = $state("");
  let userGuess = $state("");
  let accuracy = $state(0);
  let showResult = $state(false);

  // Function to encode/decode ROT13
  function rot13(str: string) {
    return str.replace(/[a-zA-Z]/g, function (chr) {
      const start = chr <= "Z" ? 65 : 97;
      return String.fromCharCode(
        start + ((chr.charCodeAt(0) - start + 13) % 26)
      );
    });
  }

  // Function to calculate accuracy
  function calculateAccuracy(original: string, guess: string) {
    let correct = 0;
    const originalWords = original.toLowerCase().split(" ");
    const guessWords = guess.toLowerCase().split(" ");

    for (
      let i = 0;
      i < Math.min(originalWords.length, guessWords.length);
      i++
    ) {
      if (originalWords[i] === guessWords[i]) correct++;
    }

    return Math.round((correct / originalWords.length) * 100);
  }

  // Function to generate a random sentence
  async function generateSentence() {
    const response = await fetch("https://api.quotable.io/random");
    const data = await response.json();
    sentence = data.content;
    encodedSentence = rot13(sentence);
    userGuess = "";
    showResult = false;
  }

  // Function to check the user's guess
  function checkGuess() {
    accuracy = calculateAccuracy(sentence, userGuess);
    showResult = true;
  }

  onMount(() => {
    generateSentence();
  });
</script>

<main class="container mx-auto p-4">
  <h1 class="text-2xl font-bold mb-4">ROT13 Guessing Game</h1>

  <div class="mb-4">
    <p class="font-semibold">Encoded sentence:</p>
    <p class="bg-gray-100 p-2 rounded">{encodedSentence}</p>
  </div>

  <div class="mb-4">
    <label for="guess" class="block font-semibold">Your guess:</label>
    <textarea
      id="guess"
      bind:value={userGuess}
      class="w-full p-2 border rounded"
      rows="3"
    ></textarea>
  </div>

  <button
    onclick={checkGuess}
    class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600"
  >
    Check Guess
  </button>

  {#if showResult}
    <div class="mt-4">
      <p class="font-semibold">Original sentence:</p>
      <p class="bg-gray-100 p-2 rounded">{sentence}</p>
      <p class="mt-2">Your accuracy: {accuracy}%</p>
    </div>
  {/if}

  <button
    onclick={generateSentence}
    class="mt-4 bg-green-500 text-white px-4 py-2 rounded hover:bg-green-600"
  >
    New Sentence
  </button>
</main>

<style>
  :global(body) {
    font-family: Arial, sans-serif;
  }
</style>
