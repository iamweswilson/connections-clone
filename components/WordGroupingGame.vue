<template>
  <div class="word-grouping-game">
    <p class="instructions">Create four groups of four!</p>
    <div class="word-grid">
      <!-- Render solved words first -->
      <button
        v-for="word in solvedWords"
        :key="word.id"
        class="solved"
        :style="{ backgroundColor: word.color }"
        disabled
      >
        {{ word.word }}
      </button>
      <!-- Render unsolved words -->
      <button
        v-for="word in unsolvedWords"
        :key="word.id"
        @click="handleWordClick(word)"
        :class="{ selected: selectedWords.includes(word) }"
        :disabled="solvedCategories.includes(word.category)"
      >
        {{ word.word }}
      </button>
    </div>
    <div class="controls">
      <button @click="checkSelection" :disabled="selectedWords.length !== 4">Submit</button>
      <button @click="shuffleWords">Shuffle</button>
    </div>
    <div v-if="message" class="message" :class="{ success: isSuccess }">
      {{ message }}
    </div>
    <div class="solved-categories">
      <h2>Uncovered Hints:</h2>
      <ul>
        <li v-for="category in solvedCategories" :key="category">{{ category }}</li>
      </ul>
    </div>
    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="#4CAF50" class="success-icon">
          <path stroke-linecap="round" stroke-linejoin="round" d="M16.5 18.75h-9m9 0a3 3 0 0 1 3 3h-15a3 3 0 0 1 3-3m9 0v-3.375c0-.621-.503-1.125-1.125-1.125h-.871M7.5 18.75v-3.375c0-.621.504-1.125 1.125-1.125h.872m5.007 0H9.497m5.007 0a7.454 7.454 0 0 1-.982-3.172M9.497 14.25a7.454 7.454 0 0 0 .981-3.172M5.25 4.236c-.982.143-1.954.317-2.916.52A6.003 6.003 0 0 0 7.73 9.728M5.25 4.236V4.5c0 2.108.966 3.99 2.48 5.228M5.25 4.236V2.721C7.456 2.41 9.71 2.25 12 2.25c2.291 0 4.545.16 6.75.47v1.516M7.73 9.728a6.726 6.726 0 0 0 2.748 1.35m8.272-6.842V4.5c0 2.108-.966 3.99-2.48 5.228m2.48-5.492a46.32 46.32 0 0 1 2.916.52 6.003 6.003 0 0 1-5.395 4.972m0 0a6.726 6.726 0 0 1-2.749 1.35m0 0a6.772 6.772 0 0 1-3.044 0" />
        </svg>
        <h2>Congratulations!</h2>
        <p>You've uncovered all of the hints... Send a screenshot of this to Dad and Cat, but not your siblings</p>
        <button @click="closeModal">Close</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';

const words = [
  { id: 1, word: 'JALISCO', category: 'Puerto Vallarta', color: '#FFA500' },
  { id: 2, word: 'MALECÓN', category: 'Puerto Vallarta', color: '#FFA500' },
  { id: 3, word: 'PLAYAS', category: 'Puerto Vallarta', color: '#FFA500' },
  { id: 4, word: 'BANDERAS BAY', category: 'Puerto Vallarta', color: '#FFA500' },
  { id: 5, word: 'TREE', category: 'Christmas', color: '#008000' },
  { id: 6, word: 'STOCKINGS', category: 'Christmas', color: '#008000' },
  { id: 7, word: 'ORNAMENTS', category: 'Christmas', color: '#008000' },
  { id: 8, word: 'POINSETTIA', category: 'Christmas', color: '#008000' },
  { id: 9, word: 'TOILETRIES', category: 'Travel Essentials', color: '#0000FF' },
  { id: 10, word: 'PASSPORT', category: 'Travel Essentials', color: '#0000FF' },
  { id: 11, word: 'SUNSCREEN', category: 'Travel Essentials', color: '#0000FF' },
  { id: 12, word: 'SUITCASE', category: 'Travel Essentials', color: '#0000FF' },
  { id: 13, word: 'NICE', category: 'Wilsons', color: '#800080' },
  { id: 14, word: 'FAMILY', category: 'Wilsons', color: '#800080' },
  { id: 15, word: 'FUN', category: 'Wilsons', color: '#800080' },
  { id: 16, word: 'HOUSE', category: 'Wilsons', color: '#800080' },
];

const gameWords = ref([...words]);
const selectedWords = ref([]);
const message = ref('');
const isSuccess = ref(false);
const solvedCategories = ref([]);
const solvedWords = ref([]);

const unsolvedWords = computed(() => {
  return gameWords.value.filter(word => !solvedWords.value.includes(word));
});

const shuffleWords = () => {
  gameWords.value = [...unsolvedWords.value].sort(() => Math.random() - 0.5);
};

const handleWordClick = (word) => {
  if (solvedCategories.value.includes(word.category)) return;
  
  const index = selectedWords.value.indexOf(word);
  if (index === -1) {
    if (selectedWords.value.length < 4) {
      selectedWords.value.push(word);
    }
  } else {
    selectedWords.value.splice(index, 1);
  }
};

const showModal = ref(false);

const checkSelection = () => {
  if (selectedWords.value.length !== 4) return;

  const category = selectedWords.value[0].category;
  const isCorrect = selectedWords.value.every(word => word.category === category);

  if (isCorrect) {
    message.value = `Correct! You found the "${category}" category.`;
    isSuccess.value = true;
    solvedCategories.value.push(category);
    // Move solved words to the top
    solvedWords.value.push(...selectedWords.value);
    // Remove solved words from gameWords
    gameWords.value = gameWords.value.filter(word => !selectedWords.value.includes(word));
    selectedWords.value = [];
  } else {
    message.value = "Incorrect. Try again!";
    isSuccess.value = false;
    selectedWords.value = [];
  }

  setTimeout(() => {
    message.value = '';
  }, 3000);

  if (solvedCategories.value.length === 4) {
    message.value = "Congratulations! You've completed the puzzle!";
    isSuccess.value = true;
    showModal.value = true;
  }
};

const closeModal = () => {
  showModal.value = false;
};

onMounted(() => {
  shuffleWords();
});
</script>

<style scoped>
.word-grouping-game {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}
.instructions {
  text-align: center;
  font-size: 1em;
  margin-bottom: 20px;
}
.word-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: center;
  margin-bottom: 20px;
}

.word-grid button {
  flex: 1 0 21%; /* This will create a 4-column layout on larger screens */
  max-width: calc(25% - 10px); /* Ensure buttons don't grow too large */
  aspect-ratio: 2 / 1; /* Make buttons square */
  padding: 5px;
  font-size: 14px;
  font-weight: bold;
  cursor: pointer;
  border-radius: 6px;
  border: 0;
  background-color: #efefe6;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  word-break: break-word;
  overflow: hidden;
}

@media (max-width: 500px) {
  .word-grid button {
    flex: 1 0 46%; /* This will create a 2-column layout on smaller screens */
    max-width: calc(50% - 10px);
    font-size: 12px;
  }
}

button.selected {
  background-color: #5a594e;
  color: white;
}

button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.word-grid button.solved {
  color: white;
  cursor: default;
}

.controls {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

.controls button {
  font-size: 16px;
  font-weight: 600;
  padding: 10px;
  margin: 0 5px;
  width: 120px;
  border-radius: 32px;
  line-height: 1.5em;
  cursor: pointer;
  border: 1px solid;
  background-color: #fff;
  color: #000;
  border-color: #000;
}

.message {
  padding: 10px;
  margin-bottom: 20px;
  border-radius: 4px;
  text-align: center;
}

.message.success {
  background-color: #4CAF50;
  color: white;
}

.message:not(.success) {
  background-color: #f44336;
  color: white;
}

.solved-categories {
  margin-top: 20px;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  margin-bottom: 5px;
}
.success-icon {
  width: 48px;
  height: 48px;
  
}

.modal {
  position: fixed;
  z-index: 1;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgba(0,0,0,0.4);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: #fefefe;
  padding: 20px;
  border-radius: 5px;
  text-align: center;
}

.modal-content button {
  margin-top: 10px;
  padding: 5px 10px;
  cursor: pointer;
}
</style>