<script setup>
import axios from 'axios';
import { nextTick, ref, watch } from 'vue';
import CharacterCard from '../components/CharacterCard.vue';
import PaginationBar from '../components/PaginationBar.vue';
let characters = ref([]);
let currentPage = ref(1);
let characterList = ref(null);
let info = ref({
    count: 0,
    pages: 0,
    next: null,
    prev: null,
});

await getCharacters(1);

async function getCharacters(page) {
    const res = await axios.get('https://rickandmortyapi.com/api/character', {
        params: { page },
    });
    characters.value = res.data.results;
    info.value = res.data.info;
    currentPage.value = page;
}

async function changePage(page) {
    if (page < 1 || page > info.value.pages || page === currentPage.value) return;
    await getCharacters(page);
}

watch(currentPage, async () => {
    await nextTick();
    characterList.value?.scrollIntoView({ behavior: 'smooth', block: 'start' });
});

</script>
<template>
    <div ref="characterList" class="character-list">
        <PaginationBar
            :current-page="currentPage"
            :total-pages="info.pages"
            @page-change="changePage"
        />
        <div class="columns is-multiline">
        <div class="column is-3" v-for="character in characters" :key="character.id">
            <CharacterCard :character="character"></CharacterCard>
        </div>
        </div>
        <PaginationBar
            :current-page="currentPage"
            :total-pages="info.pages"
            @page-change="changePage"
        />
    </div>
</template>

<style scoped>
.character-list {
    scroll-margin-top: 1rem;
}
</style>
