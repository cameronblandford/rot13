<script lang="ts">
  import { onMount } from "svelte";

  let sentence = $state("");
  let encodedSentence = $state("");
  let userGuess = $state("");
  let accuracy = $state(0);
  let showResult = $state(false);
  let isPracticingEncoding = $state(false);

  // Function to encode/decode ROT13
  function rot13(str: string) {
    return str.replace(/[a-zA-Z]/g, function (chr) {
      const start = chr <= "Z" ? 65 : 97;
      return String.fromCharCode(
        start + ((chr.charCodeAt(0) - start + 13) % 26)
      );
    });
  }

  const toggleMode = (val: boolean) => {
    isPracticingEncoding = val;
    userGuess = "";
    showResult = false;
    generateSentence();
  };
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
  const generateSentence = async () => {
    const response = await fetch("https://api.quotable.io/random");
    const data = await response.json();
    if (isPracticingEncoding) {
      sentence = rot13(data.content);
    } else {
      sentence = data.content;
    }
    encodedSentence = rot13(sentence);
    userGuess = "";
    showResult = false;
  };

  // Function to check the user's guess
  function checkGuess() {
    accuracy = calculateAccuracy(sentence, userGuess);
    showResult = true;
  }

  //   add keyboard listener for enter key and check guess on enter
  const handleKeydown = (e: KeyboardEvent) => {
    if (e.key === "Enter") {
      e.preventDefault();
      checkGuess();
    }
  };
  onMount(() => {
    generateSentence();
    window.addEventListener("keydown", handleKeydown);
  });
</script>

<main
  class="container mx-auto p-4 font-mono max-w-[700px] min-h-screen flex flex-col"
>
  <div class="grow">
    <div class="flex gap-8 justify-between">
      <h1 class="text-3xl font-bold mb-4">the rot13 dojo</h1>
      <!-- radio button for toggling between encoding and decoding -->
      <div class="mb-4 flex flex-col text-sm text-slate-500">
        <label for="encoding" class="flex gap-2">
          <input
            type="radio"
            id="encoding"
            bind:group={isPracticingEncoding}
            value={true}
            onchange={() => toggleMode(true)}
          />Encoding</label
        >

        <label for="decoding" class="flex gap-2">
          <input
            type="radio"
            id="decoding"
            bind:group={isPracticingEncoding}
            value={false}
            onchange={() => toggleMode(false)}
          />Decoding</label
        >
      </div>
    </div>

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

    <button onclick={checkGuess} class="btn"> Check Guess </button>
    <button onclick={generateSentence} class="btn"> New Sentence </button>
    {#if showResult}
      <div class="mt-4">
        <p class="font-semibold">Original sentence:</p>
        <p class="bg-gray-100 p-2 rounded">{sentence}</p>
        <p class="mt-2">Your accuracy: {accuracy}%</p>
      </div>
    {/if}
  </div>
  <div class="text-xs text-slate-500 mt-4">
    for feature suggestions or feedback, dm me on <a
      href="https://bsky.app/profile/cam.fyi"
      target="_blank"
      class="underline hover:font-bold"
    >
      bsky
    </a>
  </div>
</main>

<style lang="postcss">
  .btn {
    @apply border px-4 py-2 hover:bg-gray-100 rounded;
  }
</style>
