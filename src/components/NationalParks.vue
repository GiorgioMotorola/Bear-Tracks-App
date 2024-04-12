<template>
    <div>
        <h1 class="title">Search for National Parks</h1>
        <div class="search-container">
            <input class="search" type="text" v-model="searchQuery" placeholder="Enter state name or abbreviation"
                @keyup="handleAutocomplete">
            <div class="results-container" v-if="suggestions.length > 0">
                <ul>
                    <li v-for="(suggestion, index) in suggestions" :key="index" @click="selectSuggestion(suggestion)">{{
                suggestion }}</li>
                </ul>
            </div>
            <button @click="searchParks">Search</button>
        </div>

        <ul v-if="parks.length > 0">
            <li v-for="park in parks" :key="park.id">
                <h2>{{ park.name }}</h2>
                <ul>
                    <li v-for="activity in park.activities" :key="activity.id">
                        {{ activity.name }}
                    </li>
                </ul>
                <ul>
                    <li v-for="image in park.images" :key="image.url">
                        <img :src="image.url" :alt="park.name + ' image'" style="max-width: 200px;">
                    </li>
                </ul>
                <p>{{ park.address }}</p>
                <p>{{ park.description }}</p>
            </li>
        </ul>
    </div>

</template>

<script>
import states from '@/components/stateFullAndAbbreviated.js';
import apiKey from '@/components/apiKey.js';

export default {
    data() {
        return {
            searchQuery: '',
            suggestions: [],
            parks: [],
        }
    },
    methods: {
        handleAutocomplete() {
            if (!this.searchQuery) {
                this.suggestions = [];
                return;
            }

            this.suggestions = Object.values(states).filter(state => {
                return state.toLowerCase().includes(this.searchQuery.toLowerCase());
            });
        },

        selectSuggestion(suggestion) {
            this.searchQuery = suggestion;
            this.suggestions = [];
        },
        async searchParks() {
            try {
                let stateCode = this.searchQuery.toUpperCase();

                if (!states[stateCode]) {
                    const stateName = Object.values(states).find(name => name.toLowerCase() === this.searchQuery.toLowerCase());
                    if (stateName) {
                        stateCode = Object.keys(states).find(key => states[key].toLowerCase() === stateName.toLowerCase());
                    } else {
                        console.error('State not found:', this.searchQuery, 'State mappings:', states);
                        this.parks = [];
                        return;
                    }
                }

                console.log('State code used:', stateCode);

                const response = await fetch(`https://developer.nps.gov/api/v1/parks?stateCode=${stateCode}&api_key=${apiKey}`);
                const data = await response.json();
                console.log('API Response:', data);
                if (data.data && data.data.length > 0) {
                    this.parks = data.data;
                } else {
                    this.parks = [];
                }
            } catch (error) {
                console.error('Error fetching parks:', error);
                this.parks = [];
            }
        }

    }
}
</script>

<style scoped>
* {
    margin: 0;
    box-sizing: border-box;
}

.title {
    display: flex;
    justify-content: center;
    margin-bottom: 5%;
}

.search-container {
    position: relative;
    display: flex;
    justify-content: center;
}

.search {
    padding: .7rem;
    margin-right: 5px;
    width: 15%;
    margin-right: 1rem;
}

button {
    padding: 0.5rem 2rem;
}

.results-container {
    position: absolute;
    top: calc(100% + 5px);
    left: 0;
    width: 100%;
    background-color: white;
    border: 1px solid #ccc;
    border-top: none;
    list-style-type: none;
    padding: 0;
    margin: 0;
    z-index: 1;
}


li {
    cursor: pointer;
}
</style>@/components/stateMappings.js@/components/stateFullAndAbbreviated.js