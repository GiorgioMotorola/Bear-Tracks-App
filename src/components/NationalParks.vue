<template>
    <div class="body">
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
                <div class="park-address">{{ park.addresses[0].city }}, {{ park.addresses[0].stateCode }}</div>
                <div class="park-description">{{ park.description }}</div>
                <img class="image" :src="park.images[0].url" :alt="park.name + ' image'" loading="lazy">
            </div>
        </div>
        <transition name="park-details">
            <div v-if="selectedPark" class="park-details">
                <button class="close-button" @click="closeParkDetails">CLOSE</button>
                <div class="details-park-name">{{ selectedPark.name }}</div>
                <div class="details-park-description">{{ selectedPark.description }}</div>
                <div class="details-email-address" v-for="emailAddress in selectedPark.contacts.emailAddresses"
                    :key="emailAddress.emailAddress">
                    {{ emailAddress.emailAddress }}
                </div>
                <div class="details-phone-numbers" v-if="selectedPark.contacts.phoneNumbers.length">
                    <div class="phone-numbers" v-for="phoneNumber in selectedPark.contacts.phoneNumbers"
                        :key="phoneNumber.phoneNumber">
                        {{ phoneNumber.type }}: {{ phoneNumber.phoneNumber }}
                    </div>

                </div>
                <div class="details-activities">
                    <div class="activities" style=" background-color: #efefef;">
                        <strong style=" background-color: #efefef;">Activities: </strong>
                        <span v-for="(activity, index) in selectedPark.activities" :key="index">
                            &#160;{{ activity.name }}{{ index !== selectedPark.activities.length - 1 ? ', ' : '' }}
                        </span>
                    </div>
                </div>
                <div class="details-directions">
                    <div v-if="selectedPark.directionsInfo">
                        <p><strong>Directions:</strong> {{ selectedPark.directionsInfo }}</p>
                    </div>
                    <div v-else>
                        <p>No directions available.</p>
                    </div>
                    <div class="park-standard-hours">
                        <h2>Standard Hours</h2>
                        <ul v-for="(hours, day) in selectedPark.operatingHours[0].standardHours" :key="day">
                            <strong>{{ day }}</strong>: {{ hours }}
                        </ul>
                    </div>
                    <div v-if="selectedPark.directionsUrl">
                        <p><strong>Directions URL:</strong> <a :href="selectedPark.directionsUrl">{{
                selectedPark.directionsUrl }}</a></p>
                    </div>
                </div>
                <div class="google-maps-link" v-if="selectedPark">
                    <a :href="googleMapsLink" target="_blank">View on Google Maps</a>
                </div>
                <div class="details-park-images">
                    <div class="details-image" v-for="image in selectedPark.images" :key="image.url"
                        @click="openModal(image.url)">
                        <img :src="image.url" :alt="selectedPark.name + ' image'">
                    </div>
                </div>
            </div>
        </transition>
        <ImageModal :image="selectedImage" v-if="selectedImage" @close="closeModal" />
    </div>
    <!-- <footer class="footer">
        <a href="https://www.flaticon.com/free-icons/christmas-tree" title="christmas tree icons"
            style="text-decoration: none; color: black;">Icons
            created by Pixel perfect - Flaticon</a>
    </footer> -->
</template>

<script>
import states from '@/components/stateFullAndAbbreviated.js';
import apiKey from '@/components/apiKey.js';
import ImageModal from '@/components/ImageModal.vue';

export default {
    data() {
        return {
            searchQuery: '',
            suggestions: [],
            parks: [],
            selectedSuggestion: null,
            selectedPark: null,
            isParkSelected: false,
            selectedImage: null,
        }
    },

    computed: {
        googleMapsLink() {
            if (this.selectedPark && this.selectedPark.latitude && this.selectedPark.longitude) {
                const latitude = this.selectedPark.latitude;
                const longitude = this.selectedPark.longitude;
                return `https://www.google.com/maps/search/?api=1&query=${latitude},${longitude}`;
            }
            return '';
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
        openModal(imageUrl) {
            this.selectedImage = imageUrl;
        },
        closeModal() {
            this.selectedImage = null;
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
    },
    components: {
        ImageModal,
    }
}
</script>

<style scoped>
* {
    margin: 0;
    box-sizing: border-box;
    font-family: 'national-park', sans-serif;
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
    color: #2C3639;
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
    position: relative;
    width: 100%;
    margin: 10px;
    border: 1px solid #000000;
    border-left: none;
    border-right: none;
    box-shadow: rgba(50, 50, 93, 0.25) 0px 2px 5px -1px, rgba(0, 0, 0, 0.3) 0px 1px 3px -1px;
    flex-direction: row;
    align-items: start;
    text-align: start;
    overflow: hidden;
    padding: 10px;
}

.park-container::before,
.park-container::after {
    content: '';
    position: absolute;
    left: 0;
    width: 0;
    height: 1px;
    background-color: #575353;
    transition: width 0.3s ease;
}

.park-container::before {
    top: 0;
}

.park-container::after {
    bottom: 0;
}

.park-container:hover::before,
.park-container:hover::after {
    width: 100%;
}

.image {
    max-width: 350px;
    box-shadow: rgba(50, 50, 93, 0.25) 0px 2px 5px -1px, rgba(0, 0, 0, 0.3) 0px 1px 3px -1px;
    border: 1px solid #000000;
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
    font-size: 15px;
    font-weight: 550;
}

.park-details-enter-active,
.park-details-leave-active {
    transition: opacity 0.5s ease-in-out, transform 0.5s ease-in-out;
}

.park-details-enter-from,
.park-details-leave-to {
    opacity: 0;
    transform: translateX(100%);
}

.park-details {
    position: fixed;
    top: 0;
    right: 0;
    height: 100%;
    width: 70%;
    overflow-y: auto;
    border: 1px solid #000000;
    background-color: #efefef;
}

.details-image {
    float: left;
    padding: 3px;
    background-color: #efefef;
}

.details-image::after {
    content: "";
    clear: both;
    display: table;
}

.details-park-images img {
    width: 150px;
    height: 150px;
    box-shadow: rgba(50, 50, 93, 0.25) 0px 2px 5px -1px, rgba(0, 0, 0, 0.3) 0px 1px 3px -1px;
    border: 1px solid #000000;
    background-color: #efefef;
    cursor: pointer;
}

.details-park-name,
.details-park-description,
.details-park-images,
.details-email-address,
.details-phone-numbers,
.details-activities,
.phone-numbers,
.activites,
p,
h2,
span,
strong,
a,
ul,
li,
.google-maps-link,
.park-standard-hours {
    background-color: #efefef;
}

.details-park-name,
.details-park-description,
.details-email-address,
.details-phone-numbers,
.phone-numbers,
h2 {
    display: flex;
    flex-direction: column
}

.details-activities {
    margin-bottom: 10px;
}

.activities {
    display: flex;
    flex-wrap: wrap;

    align-items: center;
}


.park-details.active {
    transform: translateX(0);
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
    color: #000000;
    background-color: #ffffff;
}

.search:focus {
    border: 3px solid #000000;
}

button {
    padding: .7rem;
    width: 7rem;
    border-top-right-radius: 4px;
    border-left: none;
    color: #000000;
    background-color: #ffffff;
    cursor: pointer;
}

.close-button {
    width: 100%;
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

@media screen and (max-width: 650px) {
    .park-container {
        width: 100%;
    }

    .image {
        max-width: 100%;
    }

    .park-name {
        font-size: 25px;
        font-weight: 700;
    }

    .park-address {
        font-size: 15px;
        font-style: italic;
    }

    .park-details-enter-from,
    .park-details-leave-to {
        transform: translateY(100%);
    }

    .park-details {
        bottom: 0;
        left: 0;
        width: 100%;
        height: 100vh;
    }

    .park-details.active {
        transform: translateY(0);
    }

    .park-container {
        margin-left: 0rem;
        margin-right: 0rem;
    }
}
</style>
