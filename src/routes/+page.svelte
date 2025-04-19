<script lang="ts">
    import { writable } from 'svelte/store'; // Import writable for store creation
    import { generateMaintainedQuiz, generateQuiz, type QuizQuestion } from "$lib";
    import { fade, fly } from "svelte/transition";

    let testCode = ''; // User input for the test code
    let submitted = false; // Tracks whether the form has been submitted
    let quiz: QuizQuestion[] = []; // Stores the generated quiz items
    let crossedOutItems: boolean[] = []; // Tracks which items are crossed out
    let inputError = writable(""); // Store to track validation errors

    function validateCode(code: string): boolean {
        const trimmedCode = code.trim(); // Trim whitespace from input
        const isValid = /^\d{6,7}$/.test(trimmedCode); // Validate input as 6-7 numeric digits

        if (trimmedCode === "") {
            inputError.set("Please enter a valid code"); // Show this message for empty input
        } else if (!isValid) {
            inputError.set("Code must be 6 or 7 numeric digits"); // Show this message for invalid format
        } else {
            inputError.set(""); // Clear error if valid
        }

        return isValid;
    }

    function handleSubmit() {
        if (!validateCode(testCode)) return; // Validate before proceeding

        inputError.set(""); // Clear any previous error messages

        let setSize;
        let maintainCards;
        let randomSeed;

        if (testCode.length === 7) {
            setSize = Number(testCode.slice(0, 2));
            maintainCards = testCode[2] === "1";
            randomSeed = testCode.slice(3);
        } else {
            setSize = Number(testCode[0]);
            maintainCards = testCode[1] === "1";
            randomSeed = testCode.slice(2);
        }
        if (maintainCards) {
            quiz = generateMaintainedQuiz(setSize, randomSeed)
        } else {
            quiz = generateQuiz(setSize, randomSeed)
        }
        crossedOutItems = Array(quiz.length).fill(false); // Initialize all items as not crossed out
        submitted = true; // Mark form as submitted
    }

    function toggleCrossOut(index: number) {
        crossedOutItems[index] = !crossedOutItems[index]; // Toggle cross-out state for the item
    }
</script>

{#if !submitted}
    <div class="min-h-screen flex items-center justify-center bg-gradient-to-r from-blue-100 to-purple-100 p-4">
        <form
                on:submit|preventDefault={handleSubmit}
                class="bg-white shadow-lg rounded-lg p-8 w-full max-w-md border border-gray-100"
                transition:fade
        >
            <h2 class="text-2xl font-bold mb-6 text-center text-indigo-800 border-b pb-3">
                Spondee Test Generator
            </h2>

            <div class="mb-6">
                <label for="userInput" class="block text-gray-700 text-sm font-bold mb-2">
                    Enter Spondee Test Code From the App
                </label>
                <input
                        type="text"
                        id="userInput"
                        bind:value={testCode}
                        placeholder="Enter your code here..."
                        class="w-full px-4 py-3 border {$inputError ? 'border-red-500' : 'border-gray-300'} rounded-md
                     focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent shadow-sm"
                        required
                        maxlength="7"
                />
                {#if $inputError}
                    <p class="text-red-500 text-sm mt-1 animate-pulse">{$inputError}</p>
                {:else}
                    <p class="text-xs text-gray-500 mt-1">Enter a valid code</p>
                {/if}
            </div>

            <button
                    type="submit"
                    class="w-full bg-indigo-600 text-white py-3 rounded-md hover:bg-indigo-700 transition duration-300
                   focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-opacity-50 font-semibold"
            >
                Generate Test
            </button>
        </form>
    </div>
{:else}
    <div class="min-h-screen bg-gradient-to-r from-blue-100 to-purple-100 py-12 px-4">
        <div class="container mx-auto max-w-4xl">
            <div class="bg-white rounded-xl shadow-lg p-6 md:p-8">
                <h1 class="text-3xl font-bold text-center text-indigo-800 mb-2">Spondee Test Items</h1>
                <p class="text-center text-gray-600 mb-8 border-b pb-4">This key contains 100 words, but you may stop at any time.<br>
                For each round, read the word aloud and mark it complete once the patient responds.<br>
                When finished, click “End Session” on the patient’s screen.</p>

                <div class="space-y-4">
                    {#each quiz as quizQuestion, index}
                        <div
                                on:click={() => toggleCrossOut(index)}
                                class="group relative overflow-hidden
                            {crossedOutItems[index]
                                ? 'bg-green-50 border-green-200 opacity-75'
                                : 'bg-gray-50 hover:bg-indigo-50 border-gray-200'}
                            p-4 rounded-lg cursor-pointer flex items-center transition-all duration-200 border"
                                in:fly={{ y: 20, duration: 300, delay: index * 50 }}
                        >
                            <!-- Checkmark indicator -->
                            <div class="absolute right-4 top-1/2 -translate-y-1/2">
                                {#if crossedOutItems[index]}
                                    <svg class="w-6 h-6 text-green-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/>
                                    </svg>
                                {/if}
                            </div>

                            <!-- Number badge -->
                            <div class="{crossedOutItems[index] ? 'bg-green-500' : 'bg-indigo-600'} text-white rounded-full w-8 h-8 flex items-center justify-center font-bold mr-4 flex-shrink-0 transition-colors">
                                {index + 1}
                            </div>

                            <!-- Quiz word -->
                            <p class="{crossedOutItems[index] ? 'line-through text-gray-500' : 'text-gray-800'} text-lg">
                                {quizQuestion.correctAnswer}
                            </p>
                        </div>
                    {/each}
                </div>

                <div class="mt-10 text-center">
                    <button
                            on:click={() => submitted = false}
                            class="bg-indigo-600 text-white py-2 px-6 rounded-md hover:bg-indigo-700 transition duration-300
                           focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-opacity-50 font-medium"
                    >
                        Create New Test
                    </button>
                </div>
            </div>
        </div>
    </div>
{/if}
