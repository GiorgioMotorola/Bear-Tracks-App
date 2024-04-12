<template>
    <div>
        <h1 class="title">Search for National Parks</h1>
        <div class="search-container">
            <input class="search" type="text" v-model="searchQuery" placeholder="Enter state name or abbreviation"
                @keyup="handleAutocomplete" @keyup.enter="searchParks">
            <button @click="searchParks">Search</button>
        </div>
        <div class="results-container" v-if="suggestions.length > 0">
            <div class="result-text">
                <h4 v-for="(suggestion, index) in suggestions" :key="index" @click="selectSuggestion(suggestion)"
                    :class="{ 'highlighted': suggestion === selectedSuggestion }">
                    {{ suggestion }}
                </h4>
            </div>

        </div>

        <div v-if="parks.length > 0">
            <div v-for="park in parks" :key="park.id">
                <h2>{{ park.name }}</h2>
                <img :src="park.images[0].url" :alt="park.name + ' image'" style="max-width: 500px;">
                <!-- <ul>
                    <li v-for="activity in park.activities" :key="activity.id">
                        {{ activity.name }}
                    </li>
                </ul>
                <ul>
                    <li v-for="image in park.images" :key="image.url">
                        <img :src="image.url" :alt="park.name + ' image'" style="max-width: 200px;">
                    </li>
                </ul> -->
                <p>{{ park.address }}</p>
                <p>{{ park.description }}</p>
            </div>
        </div>
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
            selectedSuggestion: null,
        }
    },
    methods: {
        handleAutocomplete(event) {
            if (!this.searchQuery) {
                this.suggestions = [];
                return;
            }
            this.suggestions = Object.values(states).filter(state => {
                return state.toLowerCase().includes(this.searchQuery.toLowerCase());
            });
            if (event.key === 'ArrowUp' || event.key === 'ArrowDown') {
                event.preventDefault();

                if (this.suggestions.length > 0) {
                    const currentIndex = this.suggestions.indexOf(this.selectedSuggestion);
                    let newIndex;

                    if (event.key === 'ArrowUp') {
                        newIndex = currentIndex > 0 ? currentIndex - 1 : this.suggestions.length - 1;
                    } else {
                        newIndex = currentIndex < this.suggestions.length - 1 ? currentIndex + 1 : 0;
                    }

                    this.selectedSuggestion = this.suggestions[newIndex];
                }
            } else if (event.key === 'Enter' && this.selectedSuggestion !== null) {
                this.searchQuery = this.selectedSuggestion;
                this.suggestions = [];
                this.searchParks();
            }
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
    width: 20rem;
    border-top-left-radius: 4px;

}

button {
    padding: .7rem;
    width: 7rem;
    border-top-right-radius: 4px;
    border-left: none;
}

.search:focus {
    outline: none;
}

.results-container {
    margin: 0 auto;
    width: 27rem;
    border-bottom-left-radius: 4px;
    border-bottom-right-radius: 4px;
    background-color: #fff;
    border: 1px solid #000000;
    box-shadow: rgba(50, 50, 93, 0.25) 0px 2px 5px -1px, rgba(0, 0, 0, 0.3) 0px 1px 3px -1px;
    cursor: pointer;
    font-weight: 900;
    border-top: none;
}

.result-text {
    text-indent: 10px;
    padding: 1px;
    font-size: 20px;
}

.highlighted {
    background-color: #f0f0f0;
}
</style>
