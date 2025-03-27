<script lang="ts">
  import { generateQuiz, type QuizQuestion } from "$lib";
  let testCode = '';
  let submitted = false;
  let quiz: QuizQuestion[];
  
  
  function handleSubmit() {
      // Add your submit logic here
      console.log('Submitted:', testCode);
      let setSize;
        let maintainCards;
        let randomSeed;
        if (testCode.length === 7) {
          setSize = Number(testCode.slice(0, 2));
          maintainCards = testCode[2] === "1" ? true : false;
          randomSeed = testCode.slice(3);
        } else {
          setSize = Number(testCode[0]);
          maintainCards = testCode[2] === "1" ? true : false;
          randomSeed = testCode.slice(2);
        }

      quiz = generateQuiz(setSize, randomSeed);
      submitted = true;
  }
</script>

{#if !submitted}
<div class="min-h-screen flex items-center justify-center bg-gray-100 p-4">
  <form 
      on:submit|preventDefault={handleSubmit} 
      class="bg-white shadow-md rounded-lg p-8 w-full max-w-md"
  >
      <h2 class="text-2xl font-bold mb-6 text-center text-gray-800">
          Submit Form
      </h2>
      
      <div class="mb-4">
          <label 
              for="userInput" 
              class="block text-gray-700 text-sm font-bold mb-2"
          >
              Enter Your Message
          </label>
          <input 
              type="text" 
              id="userInput"
              bind:value={testCode}
              placeholder="Type something..."
              class="w-full px-3 py-2 border border-gray-300 rounded-md 
                     focus:outline-none focus:ring-2 focus:ring-blue-500 
                     focus:border-transparent"
          />
      </div>
      
      <button 
          type="submit" 
          class="w-full bg-blue-500 text-white py-2 rounded-md 
                 hover:bg-blue-600 transition duration-300 
                 focus:outline-none focus:ring-2 focus:ring-blue-500 
                 focus:ring-opacity-50"
      >
          Submit
      </button>
  </form>
</div>
{:else}
<div class="container mx-auto px-4 py-8">
  <div class="grid grid-cols-1 gap-4">
      {#each quiz as quizQuestion}
          <div class="bg-white shadow-md rounded-lg p-4 text-center hover:bg-gray-100 transition duration-300">
              {quizQuestion.correctAnswer}
          </div>
      {/each}
  </div>
</div>
{/if}