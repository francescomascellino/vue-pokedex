<script>

export default {
    name: "SearchBar",

    props: { query: String },

    emits: ['update:query', 'startSearch']

}

</script>

<template>

    <div class="d-flex align-items-center">

        <input type="search" name="search" id="search" class="text-light rounded bg-tertiary bg-opacity-50 w-75 mx-1"
            placeholder="Search for a Pokémon!" :value="query" @input="$emit('update:query', $event.target.value)"
            @keypress.enter="$emit('startSearch')">

        <button class="rounded-circle bg-danger border-1 me-1" @click="$emit('startSearch')"><i
                class="fa-solid fa-magnifying-glass"></i></button>

    </div>


</template>

<style scoped>
button {
    height: 25px;
    width: 25px;
}

input[type=search] {
    height: 25px;
    font-size: small;
    padding-left: 8px;
}
</style>