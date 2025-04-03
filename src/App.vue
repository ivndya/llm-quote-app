<template>
  <div class="min-h-screen flex flex-col items-center justify-center bg-gray-100 p-4">
    <div class="bg-white shadow-lg rounded-lg p-6 max-w-lg text-center">
      <p class="text-xl font-semibold">{{ quote }}</p>
      <p class="text-gray-500 mt-2">- {{ author }}</p>
      <button @click="fetchQuote" class="mt-4 bg-blue-500 text-white px-4 py-2 rounded cursor-pointer">
        Новая цитата
      </button>
      <button @click="saveFavorite" :disabled="isFavorite"
        class="mt-2 ml-2 bg-green-500 text-white px-4 py-2 rounded cursor-pointer disabled:opacity-50">
        {{ isFavorite ? 'Уже в избранном' : 'Добавить в избранное' }}
      </button>
      <button @click="explainQuote" class="mt-2 ml-2 bg-purple-500 text-white px-4 py-2 rounded">
        Объяснить смысл
      </button>
      <button @click="translateQuote" class="mt-2 ml-2 bg-yellow-500 text-white px-4 py-2 rounded">
        Перевести цитату
      </button>
      <p v-if="isGenerating" class="mt-4 text-gray-500">Генерация объяснения...</p>
      <p v-if="explanation" class="mt-4 p-4 bg-gray-200 rounded">{{ explanation }}</p>
    </div>
    <div v-if="favorites.length" class="mt-6 w-full max-w-lg">
      <h2 class="text-lg font-bold">Избранные цитаты:</h2>
      <ul>
        <li v-for="(fav, index) in favorites" :key="index" class="mt-2 p-2 border rounded bg-white shadow">
          <div>
            <p>{{ fav.text }}</p>
            <p class="text-gray-500">- {{ fav.author }}</p>
          </div>
          <button @click="removeFavorite(index)"
            class="bg-red-500 text-white px-2 py-1 rounded cursor-pointer">Удалить</button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue';

export default {
  setup() {
    const quote = ref('');
    const author = ref('');
    const favorites = ref(JSON.parse(localStorage.getItem('favorites')) || []);
    const explanation = ref('');
    const isGenerating = ref(false);

    const fetchQuote = async () => {
      const res = await fetch('https://dummyjson.com/quotes/random');
      const data = await res.json();
      quote.value = data.quote;
      author.value = data.author;
      explanation.value = '';
    };

    const saveFavorite = () => {
      if (!isFavorite.value) {
        favorites.value.push({ text: quote.value, author: author.value });
        localStorage.setItem('favorites', JSON.stringify(favorites.value));
      }
    };

    const removeFavorite = (index) => {
      favorites.value.splice(index, 1);
      localStorage.setItem('favorites', JSON.stringify(favorites.value));
    }

    const explainQuote = async () => {
      isGenerating.value = true;
      const res = await fetch('http://localhost:11434/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          model: 'command-r7b',
          prompt: `Объясни смысл цитаты: "${quote.value}". Отвечай кратко.`,
          stream: false
        })
      });
      const data = await res.json();
      explanation.value = data.response;
      isGenerating.value = false;
    };

    const translateQuote = async () => {
      const res = await fetch('http://localhost:11434/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          model: 'command-r7b',
          prompt: `Переведи на русский язык: "${quote.value}"`,
          stream: false
        })
      });
      const data = await res.json();
      quote.value = data.response;
    };

    const isFavorite = computed(() => {
      return favorites.value.some(fav => fav.text === quote.value);
    })

    onMounted(fetchQuote);

    return { quote, author, fetchQuote, saveFavorite, removeFavorite, favorites, isFavorite, translateQuote, explainQuote, explanation, isGenerating };
  }
};
</script>

<style>
body {
  font-family: Arial, sans-serif;
}
</style>