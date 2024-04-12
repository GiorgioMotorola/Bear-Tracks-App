<template>
    <div>
        <div class="title">Bear Tracks
            <img src="christmas-tree.png" alt="Icons
            created by Pixel perfect - Flaticon" style="max-width: 40px;">
        </div>
        <h4 class="sub-title">Aventure is just a short hike away</h4>
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

        <div class="parks-container" v-if="parks.length > 0">
            <div v-for="park in parks" :key="park.id" class="park-container" @click="selectPark(park)">
                <div class="park-name">{{ park.name }}</div>
                <img :src="park.images[0].url" :alt="park.name + ' image'" style="max-width: 500px;">
                <div class="park-address">{{ park.addresses[0].city }}, {{ park.addresses[0].stateCode }}</div>
                <div class="park-description">{{ park.description }}</div>
            </div>
        </div>
        <transition name="park-details">
            <div v-if="selectedPark" class="park-details">
                <button class="close-button" @click="closeParkDetails">&times;</button>
                <h2>{{ selectedPark.name }}</h2>
                <p>{{ selectedPark.description }}</p>
            </div>
        </transition>

    </div>
    <footer class="footer">
        <a href="https://www.flaticon.com/free-icons/christmas-tree" title="christmas tree icons"
            style="text-decoration: none; color: black;">Icons
            created by Pixel perfect - Flaticon</a>
    </footer>
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
            selectedPark: null,
            isParkSelected: false,
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
        selectPark(park) {
            this.selectedPark = park;
            this.isParkSelected = true;
        },
        closeParkDetails() {
            this.selectedPark = null;
            this.isParkSelected = false;
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
    font-family: 'national-park', sans-serif;
}

body {
    margin: 0;
    padding: 0;
}

@font-face {
    font-family: 'national-park';
    src: url('/np-font/NationalPark-Light.otf') format('opentype'),
}

.title {
    display: flex;
    justify-content: center;
    font-size: 40px;
    margin-bottom: 5px;
    gap: 10px;
    font-weight: 800;
}

.sub-title {
    display: flex;
    justify-content: center;
    margin-bottom: 5%;
    font-style: italic;
    color: forestgreen;
}

.parks-container {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
}

.park-container {
    width: 45%;
    margin: 10px;
    border: 1px solid #000000;
    box-shadow: rgba(50, 50, 93, 0.25) 0px 2px 5px -1px, rgba(0, 0, 0, 0.3) 0px 1px 3px -1px;
    padding: 1rem;
    flex-direction: column;
    align-items: center;
    text-align: center;
}

.park-container img {
    max-width: 100%;
}

.park-name {
    font-size: 30px;
    font-weight: 800;
}

.park-address {
    font-size: 15px;
    font-style: italic;
}

.park-description {
    font-size: 18px;
    font-weight: 500;
}

.park-details-enter-active,
.park-details-leave-active {
    transition: opacity 0.5s ease-in-out, transform 0.5s ease-in-out;
}

.park-details-enter-from,
.park-details-leave-to {
    opacity: 0;
    transform: translateY(100%);
}

.park-details {
    position: fixed;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 70vh;
    overflow-y: auto;
    padding: 20px;
    border: 1px solid #ccc;
    background-color: #f9f9f9;
}

.park-details.active {
    transform: translateY(0);
    opacity: 1;
    z-index: 9999;
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

.footer {
    position: fixed;
    padding: 10px 10px 10px 10px;
    bottom: 0;
    width: 100%;
    margin: 0;
    height: 40px;
    background: rgb(255, 255, 255);
    text-align: center;
}
</style>
