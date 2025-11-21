<script setup>
    import { onMounted, ref } from 'vue';
    import redMarkerImage from "@/assets/redmarker.png"

    const props = defineProps([
        "rotation",
        "interfaceOpacity",
        "mapAfterHover"
    ]);

    const markerPlaced = ref(false);

    let guessMarker;

    const redMarker = L.icon({
        iconUrl: redMarkerImage,
        iconSize: [24, 41],
        iconAnchor: [12, 41]
    })

    onMounted(() => {
        const map = L.map('map').setView([45, 0], 1);
        L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
            maxZoom: 19,
            attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
        }).addTo(map);

        guessMarker = L.marker([], { icon: redMarker });

        map.on("click", (e) => {
            guessMarker.setLatLng(e.latlng);
            if (!markerPlaced.value) {
                guessMarker.addTo(map);
                markerPlaced.value = true;
            }
        })

        document.querySelector("#main").addEventListener("mouseenter", updateMapSize)
        document.querySelector("#main").addEventListener("mouseleave", updateMapSize)

        function updateMapSize() {
            const mapUpdater = setInterval(() => map.invalidateSize(), 10);
            setTimeout(() => clearInterval(mapUpdater), 200);
        }
    })
</script>

<template>
    <div 
        id="main"
        :style="{ opacity: interfaceOpacity / 100,
                  height: !mapAfterHover ? '350px' : '' }"
    >
        <div 
            id="map"
            :style="{ opacity: mapAfterHover ? '0' : '1' }"    
        ></div>

        <div id="bar">
            <button 
                id="guessButton"
                :disabled="!markerPlaced"
                @click="$emit('guessPlaced', guessMarker.getLatLng().lat, guessMarker.getLatLng().lng)"
                :title="markerPlaced ? '' : 'Place a marker on the map first'"
            >Guess</button>

            <img 
                src="@/assets/compass.png" 
                alt="Compass"
                title="Compass"
                id="compass"
                :style="{rotate: rotation * -1 + 'deg'}"
            >

            <button
                class="smallButton"
                title="Return to the primary location"
                @click="$emit('backToStart')"
            ><img 
                src="@/assets/back.png" 
                alt="<-"
            ></button>

            <button 
                class="smallButton"
                title="View information about PanoramaxGuessr"
                @click="$emit('showAboutInfo')"
            ><img 
                src="@/assets/info.png" 
                alt="i"
            ></button>

            <button 
                class="smallButton"
                title="Change the settings"
                @click="$emit('showSettings')"
            ><img 
                src="@/assets/settings.png" 
                alt="⚙"
            ></button>
        </div>
    </div>
</template>

<style scoped>
    #map {
        height: 100%;
        width: 100%;
        border-radius: 20px;
        box-shadow: 0 0 10px 3px rgb(0, 0, 0, 0.3);
        cursor: cell;
        background-color: #aad3df;
        transition-duration: 300ms;
    }

    #bar {
        height: 40px;
        width: 100%;
        margin-top: 8px;
        display: flex;
        justify-content: space-between;
    }

    #main {
        display: flex;
        flex-direction: column;
        align-items: start;
        position: fixed;
        right: 15px;
        bottom: 15px;
        width: 400px;
        transition-duration: 200ms;
        max-height: calc(100vh - 30px);
        max-width: calc(100vw - 30px);

        &:hover {
            opacity: 1 !important;
            #map {
                opacity: 1 !important;
            }
        }
    }

    #main:hover {
        width: 800px;
        height: 645px !important;
    }

    #guessButton {
        border: none;
        background-color: white;
        color: rgb(32, 32, 32);
        padding: 5px;
        font-size: 150%;
        border-radius: 20px;
        box-shadow: 0 0 10px 3px rgb(0, 0, 0, 0.3);
        width: calc(100% - 187.5px);
        font-family: Arial, Helvetica, sans-serif;
        font-weight: 600;
    }

    #backToStartButton {
        width: 40px;
        height: 40px;
    }

    .smallButton {
        width: 40px;
        height: 40px;
        border-radius: 20px;
        border: none;
        background-color: white;
        box-shadow: 0 0 10px 3px rgb(0, 0, 0, 0.3);
        color: rgb(32, 32, 32);
        display: flex;
        justify-content: center;
        align-items: center;
    }

    button:hover, #guessButton:hover {
        background-color: red;
        color: white;
        cursor: pointer;

        img {
            filter: brightness(10000%);
        }
    }

    #guessButton:disabled {
        background-color: rgb(200, 200, 200) !important;
        color: rgb(135, 135, 135) !important;
        cursor: not-allowed !important;
    }
    
    img {
        width: 20px;
        height: 20px;
    }

    #compass {
        width: 40px;
        height: 40px;
        box-shadow: 0 0 10px 3px rgb(0, 0, 0, 0.3);
        border-radius: 20px;
    }

    @media (orientation: portrait) {
        #main {
            right: 0;
            bottom: 0;
            height: calc(40vh - 7.5px) !important;
            width: 100vw;
            max-width: 100vw;
            background-color: rgb(32, 32, 32);
            border-bottom: rgb(32, 32, 32) solid 7.5px;
            opacity: 100% !important;
        }

        #map {
            border-radius: 0px;
            opacity: 100% !important;
        }

        #bar {
            padding: 0 7.5px;
            box-sizing: border-box;
        }
    }
    

    @media (max-width: 1000px) and (orientation: landscape) {
        #main:not(:hover) {
            max-width: 40vw;
            max-height: 55vh;
        }
    }
</style>