<script setup>
import {ref, computed, onMounted} from 'vue';

const query = ref('');
const my_anime = ref([]);
const search_results = ref([]);
const my_anime_asc = computed(() => {
  return my_anime.value.sort((a, b) => {
    console.log(a.title);
    console.log(b.title);
    return a.title.localeCompare(b.title);
  })
});

const getData = async (url) => {
  await fetch(url)
  .then(res => res.json())
  .then(res => {
    search_results.value = res.data;
    console.log(search_results.value, res.data);
  });
};

const searchAnime = async  () => {
  const url =`https://api.jikan.moe/v4/anime?q=${query.value}`;

  getData(url);

}

const handleInput = e => {
  if (!e.target.value) {
    search_results.value = [];
  }
}

const addAnime = (anime) => {
  search_results.value = [];
  query.value = '';
  my_anime.value.push({
    id: anime.mal_id,
    title: anime.title,
    img: anime.images.jpg.image_url,
    total_episodes: anime.episodes,
    watched_episodes: 0 
  });
  
  localStorage.setItem('my_anime', JSON.stringify(my_anime.value));
}

const increaseWatch = anime => {
  anime.watched_episodes ++
  localStorage.setItem('my_anime', JSON.stringify(my_anime.value));
}

const decreaseWatch = anime => {
  anime.watched_episodes --
  localStorage.setItem('my_anime', JSON.stringify(my_anime.value));
}

onMounted(() => {
  my_anime.value = JSON.parse(localStorage.getItem('my_anime')) || [];
});

</script>

<template>
  <main>
    <h1>My Anime tracker</h1>
    <form action="" @submit.prevent="searchAnime">
      <input type="text" placeholder="Search for an anime" v-model="query" @input="handleInput">
      <button type="submit" class="button">Search</button>
    </form>

    <div class="results" v-if="search_results.length > 0">
      <div class="result" v-for="anime in search_results" :key="anime.mal_id">
        <img :src="anime.images.jpg.image_url" alt="">
        <div class="details">
          <h3>{{ anime.title }}</h3>
          <p :title="anime.synopsis" v-if="anime.synopsis">
            {{ anime.synopsis.slice(0, 120) }}
          </p>
          <button @click="addAnime(anime)" class="button">Add to my anime</button>
          <span class="flex-1"></span>
        </div>
      </div>
    </div>

    <div class="my_anime" v-if="my_anime.length > 0">
      <h2>My animes</h2>
      <div v-for="anime in my_anime_asc" :key="anime.id" class="anime">
        <img :src="anime.img" alt="">
        <h3>{{ anime.title }}</h3>
        <div class="flex-1"></div>
        <span class="episodes">{{ anime.watched_episodes}} / {{anime.total_episodes}}</span>
        <button class="button" v-if="anime.total_episodes !== anime.watched_episodes" @click="increaseWatch(anime)">+</button>
        <button class="button" v-if="anime.watched_episodes > 0" @click="decreaseWatch(anime)">-</button>
      </div>
    </div>
  </main>
</template>

<style>
  * {
    margin:0;
    padding: 0;
    box-sizing: border-box;
    font-family: Montserrat, sans-serif;
  }

  body {
    background-color: #EEE;
  }

  main {
    margin: 0 auto;
    max-width: 768px;
    padding: 1.5rem;
  }

  h1 {
    text-align:center;
    margin-bottom: 1.5rem;
  }

  form {
    display: flex;
    max-width: 400px;
    margin: 0 auto 1.5rem;
  }

  form input {
    appearance: none;
    outline: none;
    border: none;
    background: white;
    display: block;
    color: #888;
    font-size: 1.125rem;
    padding: 0.5rem 1.5rem;
    width: 100%;
  }

  .button {
    outline: none;
    border: none;
    cursor: pointer;

    background: none;
    padding: 0.5rem 1.5rem;
    background-image: linear-gradient(to right, deeppink 50%, darkviolet 50%);
    background-size: 200%;
    color: #fff;
    font-size: 1.125rem;
    font-weight: 700;
    text-transform:uppercase;
    transition: 0.4s;
  }

  .button:hover {
    background-position: right;
  }

  .results {
    background-color: white;
    border-radius: 0.5rem;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    max-height: 480px;
    overflow-y: scroll;
    margin-bottom: 1.5rem;
  }

  .result {
    display: flex;
    margin: 1rem;
    padding: 1rem;
    border: 1px solid #ccc;
    border-radius: 0.5rem;
    transition: 0.4s;
  }

  .result img {
    width: 100%;
    border-radius: 1rem;
    margin-right: 1rem;
  }

  .details {
    flex: 1 1 1 0%;
    display: flex;
    align-items : flex-start;
    flex-direction: column;
  }

  .flex-1 {
    flex: 1 1 0%;
  }

  .details h3 {
    font-size: 1.125rem;
    margin-bottom: 0.5rem;
  }

  .details p {
    font-size: 0.875rem;
    line-height: 1.4;
    margin-bottom: 1.75rem;
  }

  .details .button {
    margin-left: auto;
  }

  .my_anime h2 {
    color: #888;
    font-weight: 400;
    margin-bottom: 1.5rem;
  }

  .my_anime .anime {
    display: flex;
    align-items:center;
    margin-bottom: 1.5rem;
    background-color: #fff;
    padding: 0.25rem;
    border-radius: 0.5rem;
    box-shadow: 0 2px 4px rgba(0, 0, 0 , 0.1);
  }

  .anime img {
    width: 100px;
    height: 100px;
    object-fit: cover;
    border-radius: 1rem;
    margin: 1rem;
  }

  .anime h3 {
    color: #888;
    font-size: 1.125rem;
  }

  .anime .episodes {
    margin-right: 1rem;
    color: #888
  }

  .anime button:first-of-type {
    margin-right: 1rem;
  }

  .anime button:last-of-type {
    margin-right: 0.5rem;
  }

  .my_anime .anime h3 {
    max-width: 50%;
  }

</style>
