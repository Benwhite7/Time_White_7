<template>
    <section class="interactive-section">
        <div class="container-input-options">
            <input ref="inputSearch" class="input-serach-city" @focus="showHistorial = true"
                @keyup.enter="askWeather" @input="inputVariable" :value="modelValue" placeholder="Ingresa una ciudad" />

            <transition name="fade">
                <article v-if="showHistorial" class="history-searchs-container" ref="searchsCon">
                    <div class="history-search-card" v-for="(search, index) in searchs" :key="index">
                        <option class="option-search-text" @click="changeInput(search)" :value="search">
                            {{ `${search}` }}
                        </option>
                        <button class="button_delete_search_card"
                            @click="remenberSearch.includes(search) ? deleteSearchCard(search) : changeInput(search)">
                            {{ remenberSearch.includes(search) ? '❌' : '🔍' }}
                        </button>
                    </div>
                </article>
            </transition>
        </div>

        <button class="interactive-section_button" @click="askWeather">
            Buscar
        </button>
    </section>
</template>

<script setup>
import citisSearchJSON from "../assets/ciudades_latam_y_global.json";
import { onMounted, onUnmounted, ref } from 'vue';

const emit = defineEmits(['update:modelValue']);

const props = defineProps({
    modelValue: String,
    askWeather: {
        type: Function,
        required: true
    },
});

const inputSearch = ref(null);
const searchsCon = ref(null);
const searchs = ref([]);
const remenberSearch = ref([]);
const showHistorial = ref(false);

onMounted(() => {
    document.addEventListener('click', handleClickOutside);
    let history_Searchs = localStorage.getItem("historySearchs");
    if (!history_Searchs) {
        const emptyHistory = JSON.stringify([]);
        localStorage.setItem("historySearchs", emptyHistory);
        history_Searchs = emptyHistory;
    }
    remenberSearch.value = JSON.parse(history_Searchs);
    searchs.value = JSON.parse(history_Searchs);
});

onUnmounted(() => {
    document.removeEventListener('click', handleClickOutside);
});

function inputVariable(event) {
    showHistorial.value = true;    
    emit('update:modelValue', event.target.value);
    pushSuggestions(event.target.value);
};

function changeInput(newValue) {
    emit('update:modelValue', newValue.replace(/\s*\(.*?\)/g, ''));
    showHistorial.value = false;
};

function verifyExistSearch(search) {
    if (remenberSearch.value.includes(search)) {
        let indexFound = remenberSearch.value.indexOf(search);
        remenberSearch.value.splice(indexFound, 1);
    }
};

function deleteSearchCard(city) {
    let indexFound = remenberSearch.value.indexOf(city);
    let cardFound = searchs.value.indexOf(city);
    remenberSearch.value.splice(indexFound, 1);
    searchs.value.splice(cardFound, 1);
    let historySearchJSON = JSON.stringify(remenberSearch.value);
    localStorage.setItem("historySearchs", historySearchJSON);
}

function modifiedOnCallApi(name) {
    if (remenberSearch.value.length >= 4) remenberSearch.value.pop();
    verifyExistSearch(name);
    remenberSearch.value.unshift(name);
    searchs.value = remenberSearch.value;
    let historyJSON = JSON.stringify(remenberSearch.value);
    localStorage.setItem("historySearchs", historyJSON);
}

function pushSuggestions(text) {
    if (text.length < 2) {
        searchs.value = remenberSearch.value;
        return
    };
    const results = citisSearchJSON.filter((c) => c.name.toLowerCase().includes(text.toLowerCase()));
    let names = results.filter((e) => !remenberSearch.value.includes(e.name)).map((e) => `${e.name} (${e.country})`);
    if (names.length === 0) names = [text];
    searchs.value = [...remenberSearch.value, ...names];
};


function handleClickOutside(event) {
    const elementInQuestion = searchsCon.value;
    const inputElement = inputSearch.value;
    if (elementInQuestion && !elementInQuestion.contains(event.target) && event.target !== inputElement) {
        showHistorial.value = false;
    }
}


defineExpose({
    remenberSearch,
    searchs,
    modifiedOnCallApi
})

</script>