<script setup>
    import { onMounted } from 'vue';
    import redMarkerImage from "@/assets/redmarker.png"
    import greenMarkerImage from "@/assets/greenmarker.png"
    import checkerboardMarkerImage from "@/assets/checkerboardmarker.png"

    const props = defineProps([
        "correctLocation",
        "guessLocation",
        "copyright"
    ]);

    let guessLocation = props.guessLocation;

    while (guessLocation[1] > 180) {
        guessLocation[1] -= 360;
    }

    while (guessLocation[1] < -180) {
        guessLocation[1] += 360;
    }

    const redMarker = L.icon({
        iconUrl: redMarkerImage,
        iconSize: [24, 41],
        iconAnchor: [12, 41]
    })

    const greenMarker = L.icon({
        iconUrl: localStorage.getItem("checkerboardMarker") ? checkerboardMarkerImage : greenMarkerImage,
        iconSize: [24, 41],
        iconAnchor: [12, 41]
    })

    onMounted(async () => {
        const map = L.map('map').fitBounds([props.correctLocation, props.guessLocation]);

        L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
            maxZoom: 19,
            attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
        }).addTo(map);

        const correctLocationMarker = L.marker(
            props.correctLocation,
            { icon: greenMarker }
        ).addTo(map);

        const guessLocationMarker = L.marker(
            guessLocation,
            { icon: redMarker }
        ).addTo(map);

        const line = L.polyline([
            props.correctLocation,
            guessLocation
        ], {
            color: "black"
        }).addTo(map);
    });

    function newGame() {
        window.location.reload();
    }
</script>

<template>
    <div id="map"></div>
    <button @click="newGame()">Next game</button>
    <aside id="copyright">
        <h1>Photo copyright</h1>
        <div v-for="object of props.copyright">
            <h6>Author:</h6>
            <a 
                :href="'https://api.panoramax.xyz/#focus=pic&pic=' + object.examplePhoto"
                title="Go to a photo of this author that you've seen"
            >{{ object.author }}</a>
            <h6>License:</h6>
            <a 
                :href="object.licenseLink"
                title="View the license"
            >{{ object.license }}</a>
        </div>
    </aside>
</template>

<style scoped>
    #map {
        width: 100vw;
        height: 100vh;
        background-color: #aad3df;
    }

    button {
        border: none;
        background-color: white;
        color: rgb(32, 32, 32);
        padding: 15px 65px;
        font-size: 175%;
        border-radius: 100px;
        box-shadow: 0 0 10px 3px rgb(0, 0, 0, 0.3);
        cursor: pointer;
        position: fixed;
        left: calc(50vw - 135px);
        bottom: 20px;
        z-index: 999;
        font-family: Arial, Helvetica, sans-serif;
        font-weight: 600;
    }

    button:hover {
        background-color: red;
        color: white;
    }

    #copyright {
        position: fixed;
        top: 15px;
        right: 15px;
        min-width: 210px;
        max-width: calc(100vw - 70px);
        z-index: 999;
        display: flex;
        align-items: center;
        flex-direction: column;
        background-color: rgb(255, 255, 255, 0.75);
        border-radius: 20px;
        padding: 0 15px;
        max-height: 320px;
        overflow-y: auto;
        box-shadow: 0 0 10px 3px rgb(0, 0, 0, 0.3);
        text-align: center;
    }

    #copyright > section {
        padding: 0 !important;
    }

    a {
        color: red;
        text-decoration: none;
        margin-bottom: 20px;
        display: inline-block;
    }

    h6 {
        margin: 0;
    }

    p {
        margin: 0;
        margin-bottom: 5px;
    }
</style>