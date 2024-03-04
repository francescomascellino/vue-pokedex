<script>
import axios, { Axios } from 'axios';

import SearchBar from './components/SearchBar.vue';

export default {
    name: "App",

    components: {

        SearchBar

    },

    data() {

        return {

            activePokemon: null,

            activePokemonStats: null,

            captured: [],

            searchValue: "",

            side: "front",

            loading: false,

            showAlert: false

        }

    },

    filters: {
        // FORMATS STRINGS REMOVING - AND TURNING FIRST LETTER TO UPPERCASE
        formatStatName(value) {

            console.log(value);
            return value.split('-').map(word => word.charAt(0).toUpperCase() + word.slice(1)).join(' ');

        }
    },

    methods: {

        // TRASFORMA IL PERCORSO DELL'IMMAGINE LOCALE IN UN URL
        getPlaceholderImg(url) {
            return new URL(`${url}`, import.meta.url).href
        },

        search(query) {

            console.log('Search value:', query.toLowerCase());

            console.log('Search query: https://pokeapi.co/api/v2/pokemon/' + `${query.toLowerCase()}`);

            this.loading = true;

            // IF THE SEARCH INPUT IS NOT EMPTY AND DOES NOT CONTAIN ONLY BLANK SPACES
            if (query.trim !== '') {

                // TURNS THE searchValue TO LOWERCASE AND MAKES THE API CALL
                axios.get('https://pokeapi.co/api/v2/pokemon/' + `${query.toLowerCase()}`)
                    .then(response => {


                        this.activePokemon = response.data;

                        console.log('active Pokémon:', this.activePokemon.name.charAt(0).toUpperCase() + this.activePokemon.name.slice(1), this.activePokemon);

                        // EMPTIES THE SEARCH VALUE
                        this.searchValue = "";

                        // SIMULATE 5 SEC LOADING
                        setTimeout(() => {
                            this.loading = false;
                        }, 500); // 1000 milliseconds (1 seconds)

                    }).catch((error) => {

                        // ALERTS IF THE POKEMON  IS NOT FOUND AND LOGS THE ERROR
                        alert("Pokémon not found!");
                        console.log(error);
                        this.loading = false;
                    });

            } else {

                // WARNS IN CONSOLE IF THE SEARCH FIELD IS EMPTY
                console.log('The search field cannot be empty or contain only blank spaces.');
                this.loading = false;
            }

        },

        capture(pokemon) {

            console.log("captured:", this.captured);

            // IF THE POKEMON IS NOT IN THE CAPTURED ARRAY
            if (!this.captured.includes(pokemon)) {

                // PUSHES THE  NAME OF THE FOUND POKEMON INTO THE CAPTURED ARRAY
                this.captured.push(pokemon);

                // SAVES THE CAPTURED POKEMONS LIST TO THE LOCAL STORAGE
                localStorage.setItem("captured", JSON.stringify(this.captured));
            } else {

                // ELSE WARNS IN CONSOLE THAT THE POKEMON IS ALREADY CAPTURED
                console.log(pokemon, "has already been captured");
            }

            // UPDATES THE CAPTURED POKEMON ARRAY WITH THE DATA FROM LOCAL STORAGE
            this.captured = JSON.parse(localStorage.getItem("captured"));

        },

        release(capturedPokemon) {

            // REMOVES THE  GIVEN POKEMON FROM THE LOCAL STORAGE
            localStorage.removeItem("captured", capturedPokemon.toLowerCase());

            // REMOVES THE GIVEN POKEMON FROM THE CAPTURED ARRAY
            this.captured.splice(this.captured.indexOf(capturedPokemon), 1);

        },

        changeSide() {

            console.log(this.side);
            this.side = (this.side === 'front') ? 'back' : 'front';

        },

        showCaptureAlert() {
            this.showAlert = true;
        },

        hideCaptureAlert() {
            this.showAlert = false;
        }

    },

    mounted() {

        // IF THERE IS A CAPTURED ARRAY IN LOCAL STORAGE
        if (localStorage.getItem("captured")) {

            // UPDATES THE CAPTURED POKEMON ARRAY WITH THE DATA FROM LOCAL STORAGE
            this.captured = JSON.parse(localStorage.getItem("captured"));
        } else {

            // ELSE EMPTIES THE CAPTURED ARRAY
            this.captured = [];

            console.log('captured:', this.captured);

        }

    },

    computed: {
        filterPokemonData() {

            if (this.activePokemon && this.activePokemon.stats && this.activePokemon.stats.length > 0) {
                return this.activePokemon.stats.map(stat => ({
                    name: stat.stat.name.split('-').map(word => word.charAt(0).toUpperCase() + word.slice(1)).join(' '),
                    value: stat.base_stat,
                }));
            } else {
                return [];
            }

        }
    }

}

</script>

<template>
    <!-- WRAPPER -->
    <div class="d-flex align-items-center justify-content-center min-vh-100 my-3 my-sm-0">

        <div class="wrapper container-fluid">
            <!-- LEFT COLUMN -->
            <div class="row justify-content-center m-1 d-flex flex-column flex-sm-row">

                <div
                    class="case col-12 col-sm-6 border border-danger bg-danger bg-opacity-75 rounded-top rounded-sm-left">
                    <div class="col">

                        <!-- TOP AND SEARCH BAR -->
                        <div class="d-flex align-items-center">

                            <!-- ROUNDED SCREEN -->
                            <div class="round-screen col-2 border border-success rounded-circle d-flex align-items-center justify-content-center p-3 m-3 shadow"
                                style="aspect-ratio: 1/1;">

                                <span class="capture-alert"
                                    :style="{ display: showAlert ? 'block' : 'none' }"><strong>CATCH!</strong></span>

                            </div>

                            <div class="col">

                                <!-- SEARCHBAR -->
                                <!-- Per impostazione predefinita, v-model su un componente utilizza modelValue come prop e update:modelValue come evento. Possiamo modificare questi nomi passando un argomento a v-model: -->
                                <SearchBar v-model:query="searchValue" @startSearch="search(searchValue)" />
                                <!-- In questo caso, il componente figlio dovrebbe aspettarsi una prop title ed emettere un evento update:title per aggiornare il valore nel componente genitore -->

                            </div>

                        </div>

                        <div class="col-12 p-3 d-flex justify-content-center">
                            <!-- SCREEN -->
                            <div class="screen d-flex align-items-center justify-content-center border border-success rounded shadow-lg p-2"
                                style="height: 350px; width: 350px;">

                                <template v-if="loading">

                                    <!-- LOADER -->
                                    <div class="lds-spinner">
                                        <div></div>
                                        <div></div>
                                        <div></div>
                                        <div></div>
                                        <div></div>
                                        <div></div>
                                        <div></div>
                                        <div></div>
                                        <div></div>
                                        <div></div>
                                        <div></div>
                                        <div></div>
                                    </div>

                                </template>

                                <template v-else-if="activePokemon">

                                    <!-- CAPTURE BUTTON -->
                                    <div class="capture m-o " @mouseover="showCaptureAlert"
                                        @mouseout="hideCaptureAlert">
                                        <svg xmlns="http://www.w3.org/2000/svg"
                                            class="icon icon-tabler icon-tabler-pokeball" width="30" height="30"
                                            viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" fill="none"
                                            stroke-linecap="round" stroke-linejoin="round"
                                            @click="capture(activePokemon.name)"
                                            v-if="!this.captured.includes(activePokemon.name)">
                                            <path stroke="none" d="M0 0h24v24H0z" fill="none" />
                                            <circle cx="9" cy="9" r="9" transform="translate(3 3)" />
                                            <circle cx="12" cy="12" r="3" />
                                            <path d="M3 12h6m6 0h6" />
                                        </svg>
                                    </div>


                                    <div>

                                        <!-- ACTIVE POKEMON SPRITE -->
                                        <img class="preview"
                                            :src="activePokemon.sprites.other.showdown[`${side}_default`]"
                                            :alt="activePokemon.name + `${side}`" @click="changeSide()">

                                        <p class="text-center fw-bold fs-6 my-0">
                                            {{ activePokemon.name.charAt(0).toUpperCase() + activePokemon.name.slice(1) }}
                                        </p>

                                        <!-- ACTIVE POKEMON DATA -->
                                        <div class="my-1">
                                            <span class="me-2"><strong>ID:</strong> {{ activePokemon.id }}</span>

                                            <span class="me-2"><strong>Height:</strong>
                                                {{ activePokemon.height }}</span>

                                            <span class="me-2"><strong>Weight:</strong>
                                                {{ activePokemon.weight }}</span>
                                        </div>

                                        <!-- ACTIVE POKEMON STATS -->
                                        <template v-for="stat in filterPokemonData" :key="stat.name">

                                            <div class="d-flex align-items-center mt-1">

                                                <div class="col-2 col-sm fw-bold me-2">
                                                    <span>{{ stat.name }}</span>
                                                </div>

                                                <!-- STAT PROGRESS BAR -->
                                                <div class="stat-bar d-flex">

                                                    <div class="stat-value"
                                                        :style="{ width: `${(stat.value / 255) * 200}px` }">

                                                        <span class="text-success ms-2"></span>

                                                    </div>

                                                </div>

                                            </div>

                                        </template>

                                    </div>

                                </template>

                            </div>
                        </div>

                    </div>
                </div>

                <!-- MIDDLE DIVIDER -->
                <div class="divider border border-black"></div>

                <!-- RIGHT COLUMN -->
                <div
                    class="case col-12 col-sm-4 border border-danger bg-danger bg-opacity-75 rounded-bottom rounded-sm-right py-3 d-flex justify-content-center">

                    <div class="col-12 py-3 d-flex justify-content-center">

                        <!-- SCREEN -->
                        <div class="screen side border border-success rounded p-2" style="height: 350px; width: 350px;">

                            <!-- CAPTURED POKEMONS LIST -->

                            <template v-for="pokemon in captured" :key="pokemon">

                                <div class="d-flex align-items-center border-bottom border-success">

                                    <!-- CAPTURED POKEMON NAME -->
                                    <div class="col-2 col-sm me-3">
                                        <span>
                                            <strong>{{ pokemon.charAt(0).toUpperCase() + pokemon.slice(1) }}</strong>
                                        </span>
                                    </div>

                                    <!-- ACTION BUTTONS -->
                                    <div class="col">

                                        <button class="action rounded-pill me-1  my-1"
                                            @click="search(pokemon)">Show</button>

                                        <button class="action rounded-pill me-1  my-1"
                                            @click="release(pokemon)">Release</button>

                                    </div>

                                </div>

                            </template>

                        </div>
                    </div>
                </div>

            </div>

        </div>

    </div>

</template>

<style scoped>
/* #region LOADER */
.lds-spinner {
    color: official;
    display: inline-block;
    position: relative;
    width: 80px;
    height: 80px;
}

.lds-spinner div {
    transform-origin: 40px 40px;
    animation: lds-spinner 1.2s linear infinite;
}

.lds-spinner div:after {
    content: " ";
    display: block;
    position: absolute;
    top: 3px;
    left: 37px;
    width: 6px;
    height: 18px;
    border-radius: 20%;
    background: #022402;
}

.lds-spinner div:nth-child(1) {
    transform: rotate(0deg);
    animation-delay: -1.1s;
}

.lds-spinner div:nth-child(2) {
    transform: rotate(30deg);
    animation-delay: -1s;
}

.lds-spinner div:nth-child(3) {
    transform: rotate(60deg);
    animation-delay: -0.9s;
}

.lds-spinner div:nth-child(4) {
    transform: rotate(90deg);
    animation-delay: -0.8s;
}

.lds-spinner div:nth-child(5) {
    transform: rotate(120deg);
    animation-delay: -0.7s;
}

.lds-spinner div:nth-child(6) {
    transform: rotate(150deg);
    animation-delay: -0.6s;
}

.lds-spinner div:nth-child(7) {
    transform: rotate(180deg);
    animation-delay: -0.5s;
}

.lds-spinner div:nth-child(8) {
    transform: rotate(210deg);
    animation-delay: -0.4s;
}

.lds-spinner div:nth-child(9) {
    transform: rotate(240deg);
    animation-delay: -0.3s;
}

.lds-spinner div:nth-child(10) {
    transform: rotate(270deg);
    animation-delay: -0.2s;
}

.lds-spinner div:nth-child(11) {
    transform: rotate(300deg);
    animation-delay: -0.1s;
}

.lds-spinner div:nth-child(12) {
    transform: rotate(330deg);
    animation-delay: 0s;
}

@keyframes lds-spinner {
    0% {
        opacity: 1;
    }

    100% {
        opacity: 0;
    }
}

/* #endregion LOADER */

/* #region GLOBAL RULES */
p,
span,
h1 {
    color: #022402;
}

/* #endregion GLOBAL RULES */

/* #region POKEDEX CASE */
.wrapper {
    max-width: 768px;
    font-size: xx-small;
}

.case {
    box-shadow: inset 0 0 20px rgb(161, 0, 0) !important;
}

.divider {
    background: rgb(0, 0, 0);
    background: linear-gradient(90deg, rgba(0, 0, 0, 1) 0%, rgba(57, 57, 57, 1) 50%, rgba(0, 0, 0, 1) 100%);
    min-height: 25px;
}

.round-screen {
    background: rgb(30, 227, 14);
    background: radial-gradient(circle, rgba(30, 227, 14, 1) 0%, rgba(22, 83, 19, 1) 100%);
}

.capture-alert {
    text-shadow: 0 0 1px rgba(22, 83, 19, 1);
}

.screen {
    background: rgb(30, 227, 14) !important;
    box-shadow: inset 0 0 20px rgba(22, 83, 19, 1) !important;
    position: relative;
}

.side {
    overflow-y: auto;
}

/* #endregion POKEDEX CASE */

/* #region SCROOLLBAR */
/* width */
::-webkit-scrollbar {
    width: 5px;
}

/* Handle */
::-webkit-scrollbar-thumb {
    background: rgba(22, 83, 19, 1);
    border-radius: 10px;
}

/* Handle on hover */
::-webkit-scrollbar-thumb:hover {
    background: #242424;
}

/* #endregion INPUTS */

/* #region SCREENS */
.action {
    width: 50px;
}

.capture {
    position: absolute;
    z-index: 100;
    top: 0%;
    left: 0;
    transform: translate(50%, 50%);
    border-radius: 50%;
    height: 30px;
    width: 30px;
    cursor: pointer;

    &:hover {
        box-shadow: #022402 0px 0px 5px 2px;
    }

}

.preview {
    display: block;
    margin: 0 auto;
    max-width: 150px;
}

.stat-bar {
    width: 200px;
    border: 1px solid #022402;
    border-radius: 10px;
}

.stat-value {
    background-color: #022402;
    border-radius: 10px;
}

/* #endregion SCREENS */

/* #region MEDIA QUERIES */
@media (max-width: 575.98px) {
    .divider {
        background: rgb(0, 0, 0);
        background: linear-gradient(180deg, rgba(0, 0, 0, 1) 0%, rgba(57, 57, 57, 1) 50%, rgba(0, 0, 0, 1) 100%);
    }
}

@media (min-width: 575.99px) {
    .rounded-sm-left {
        border-radius: .375rem 0 0 .375rem !important;
    }

    .rounded-sm-right {
        border-radius: 0 0.375rem 0.375rem 0 !important;
    }

    .divider {
        width: 25px;
    }
}

/* #endregion MEDIA QUERIES */
</style>
