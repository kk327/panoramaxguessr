<script setup>
    import { ref, useTemplateRef } from 'vue';
    import Panoramax from './components/Panoramax.vue';
    import LocationSelector from './components/LocationSelector.vue';
    import EndScreen from './components/EndScreen.vue';
    import About from './components/About.vue';

    const location = ref([]);
    const guessLocation = ref([]);
    const loaded = ref(false);
    const gameEnded = ref(false);
    const rotation = ref(0);
    const copyright = ref([]);
    const panoramax = useTemplateRef("panoramax");
    const successfullyLoaded = ref(false);
    const loadingDotAmount = ref(0);
    const reloading = ref(false);
    const aboutVisible = ref(false);

    function onLoad(latitude, longitude) {
        loaded.value = true;
        location.value = [latitude, longitude];
    }

    function onGameEnd(latitude, longitude) {
        gameEnded.value = true;
        guessLocation.value = [latitude, longitude];
    }

    function addCopyright(object) {
        if (!copyright.value.map((e) => e.author + e.license + e.licenseLink).includes(object.author + object.license + object.licenseLink)) {
            copyright.value.push({ author: object.author, license: object.license, licenseLink: object.licenseLink });
        }
    }

    function onSuccessfulLoad() {
        clearInterval(onFailedLoad);
        clearInterval(loadingAnimation);
        successfullyLoaded.value = true;
    }

    function animateDots() {
        if (loadingDotAmount.value == 3) {
            loadingDotAmount.value = 0;
        } else {
            loadingDotAmount.value += 1;
        }
    }

    const loadingAnimation = setInterval(animateDots, 400);
    const onFailedLoad = setInterval(() => {
        reloading.value = true;
        setTimeout(() => reloading.value = false, 100);
    }, 6000);
</script>

<template>
    <Panoramax 
        v-if="!gameEnded && !reloading"
        @loaded="(lon, lat) => onLoad(lon, lat)" 
        @rotationChanged="(newRotation) => rotation = newRotation"
        @addCopyright="(object) => addCopyright(object)"
        @successfulLoad="onSuccessfulLoad()"
        ref="panoramax"
    />
    <LocationSelector 
        v-if="loaded && !gameEnded"
        :rotation="rotation"
        @guessPlaced="onGameEnd"
        @backToStart="panoramax.goBackToStart()"
        @showAboutInfo="aboutVisible = true"
    />
    <EndScreen
        v-if="gameEnded"
        :correctLocation="location"
        :guessLocation="guessLocation"
        :copyright="copyright"
    />
    <About
        v-if="aboutVisible"
        @close="aboutVisible = false"
    />
    <section 
        v-if="!successfullyLoaded"
        id="loadingText"
    >
        <p>Loading Panoramax<br>Please wait{{ ".".repeat(loadingDotAmount) }}</p>
    </section>
</template>

<style>
    body {
        margin: 0;
        background-color: #37474f;
        color: rgb(32, 32, 32);
        font-family: Arial, Helvetica, sans-serif;
    }

    .leaflet-marker-icon {
        box-shadow: 0 0 5px 5px rgb(0, 0, 0, 0.3);
        background-color: rgb(0, 0, 0, 0.3);
        border-radius: 50px;
    }

    div[class="leaflet-control-zoom leaflet-bar leaflet-control"] {
        border-radius: 20px;
        background-color: white;
        border: none !important;
        box-shadow: 0 0 5px 3px rgb(0, 0, 0, 0.3) !important;
    }

    .leaflet-control-zoom-in,
    .leaflet-control-zoom-out {
        border-radius: 20px !important;
        border: none !important;

        span {
            display: block;
            width: 30px;
            border-radius: 15px;
        }
    }

    .leaflet-control-zoom-in:hover ,
    .leaflet-control-zoom-out:hover {
        background-color: red !important;
        color: white;
    }

    a[class="leaflet-control-zoom-in leaflet-disabled"],
    a[class="leaflet-control-zoom-out leaflet-disabled"] {
        width: 0 !important;
        cursor: not-allowed !important;

        span {
            background-color: rgb(235, 235, 235);
        }
    }

    div[class="leaflet-control-attribution leaflet-control"] {
        border-radius: 7.5px 0 0;
    }

    .gvs-mini,
    #gvs-corner-main-bottom-left, 
    #gvs-corner-main-bottom-right, 
    #gvs-widget-player,
    div[class="maplibregl-map gvs-map"],
    div[class="gvs-loader gvs-loader-visible"] > div > span {
        display: none;
    }

    #loadingText {
        position: fixed;
        top: 0;
        left: 0;
        width: 100vw;
        height: 100vh;
        display: flex;
        justify-content: center;
        align-items: center;
        font-weight: 600;
        color: white;
        font-size: 150%;
        text-align: center
    }
</style>