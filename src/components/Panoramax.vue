<script setup>
    import { onMounted, ref } from 'vue';

    const emit = defineEmits([
        "loaded",
        "rotationChanged",
        "addCopyright",
        "successfulLoad"
    ]);

    const firstPicture = ref("");
    const picturesInSequence = ref([]);
    const previousSequence = ref("");
    const pictureId = ref("");
    const automovingBackwards = ref(false);
    const automovingForward = ref(false);
    const hasLoaded = ref(false);

    let viewer;
    let instance = "";

    onMounted(async () => {
        let done = false;
        let picture = "";
        const instances = ["https://panoramax.openstreetmap.fr/api", "https://panoramax.ign.fr/api"];

        while (!done) {
            instance = instances[Math.floor(Math.random() * 2)];
            
            let attempts = 0;
            const mainApiData = await(await fetch(instance + "/users")).json();
            let userLink = "";
            while (!userLink.includes("/api/users/") && attempts < 5) {
                userLink = mainApiData.links[Math.floor(Math.random() * mainApiData.links.length)].href;
                attempts++;
            }
            if (attempts == 5) {
                continue;
            }

            attempts = 0;
            const userData = await(await fetch(userLink)).json();
            let sequence = "";
            if (!userData?.links) {
                continue;
            }
            while (!sequence.includes("/api/collections/") && attempts < 5) {
                sequence = userData.links[Math.floor(Math.random() * userData.links.length)].href;
                attempts++;
            }
            if (attempts == 5) {
                continue;
            }

            attempts = 0;
            const collectionData = await(await fetch(sequence + "/items")).json();
            picture = collectionData.features[Math.floor(Math.random() * collectionData.features.length)].id;
            
            done = true;
            firstPicture.value = picture;
            pictureId.value = picture;
            picturesInSequence.value = collectionData.features.map((e) => { return e.id });
            previousSequence.value = sequence.substring(51);
            emit("addCopyright", { author: collectionData.features[0].properties["geovisio:producer"], license: collectionData.features[0].properties.license, licenseLink: collectionData.features[0].links[4].href });
        }

        viewer = new Panoramax.Viewer(
        "viewer",
        instance,
            {
                selectedPicture: picture,
                hash: false
            }
        );
        
        emit("successfulLoad");
        viewer.addEventListener("psv:picture-loaded", loaded);
        viewer.addEventListener("select", pictureChanged);
        viewer.addEventListener("focus-changed", () => viewer.setFocus("pic"));
        viewer.addEventListener("psv:view-rotated", (viewer) => emit("rotationChanged", viewer.detail.x));
        viewer.addEventListener("sequence-playing", () => automovingForward.value = true);
        viewer.addEventListener("sequence-stopped", () => automovingForward.value = false);
    })

    function loaded(viewer) {
        emit("loaded", viewer.detail.lat, viewer.detail.lon);
        this.removeEventListener("psv:picture-loaded", loaded);
        hasLoaded.value = true;
    }

    async function pictureChanged(viewer) {
        pictureId.value = viewer.detail.picId;

        if (viewer.detail.seqId && previousSequence.value != viewer.detail.seqId) {
            previousSequence.value = viewer.detail.seqId;
            const picturesData = (await(await fetch(instance + "/collections/" + viewer.detail.seqId + "/items")).json()).features;
            picturesInSequence.value = picturesData.map((e) => { return e.id });
            emit("addCopyright", { author: picturesData[0].properties["geovisio:producer"], license: picturesData[0].properties.license, licenseLink: picturesData[0].links[4].href });
        }
    }

    function goBackToStart() {
        viewer.select("", firstPicture.value);
    }

    function playSequenceBackwards() {
        if (viewer.isSequencePlaying()) {
            viewer.stopSequence();
        }

        if (!automovingBackwards.value) {
            automovingBackwards.value = true;
            automoveBackwards();
        }
    }

    function playSequenceForward() {
        automovingBackwards.value = false;
        viewer.playSequence();
    }

    async function automoveBackwards() {
        if (picturesInSequence.value.indexOf(pictureId.value) != 0 && automovingBackwards.value) {
            const id = picturesInSequence.value[picturesInSequence.value.indexOf(pictureId.value) - 1];
            viewer.select("", id);
            pictureId.value = id;
            viewer.addEventListener(("psv:picture-loaded"), () => {
                automoveBackwards();
            }, {once: true});
        } else {
            automovingBackwards.value = false;
        }
    }

    function stopAutomoving() {
        viewer.stopSequence();
        automovingBackwards.value = false;
    }

    defineExpose({
        goBackToStart
    })
</script>

<template>
    <main id="viewer"></main>
    <section 
        id="additionalNavigation" 
        v-if="hasLoaded"
        :class="{ darkened: picturesInSequence.length == 1 }"
    >
        <button 
            class="button"
            @click="playSequenceBackwards()"
            title="Automove backwards"
            :disabled="picturesInSequence.indexOf(pictureId) == 0 || picturesInSequence.length == 1"
            :class="{ selected: automovingBackwards }"
        ><img 
            src="@/assets/automove.png" 
            alt="<<"
        ></button>

        <button 
            class="button"
            @click="stopAutomoving()"
            title="Stop automoving"
            :disabled="!(automovingBackwards || automovingForward)"
        ><img 
            src="@/assets/pause.png" 
            alt="||"
        ></button>

        <button 
            class="button"
            @click="playSequenceForward()"
            title="Automove forward"
            :disabled="picturesInSequence.indexOf(pictureId) == picturesInSequence.length - 1 || picturesInSequence.length == 1"
            :class="{ selected: automovingForward }"
        ><img 
            src="@/assets/automove.png" 
            alt="<<" 
            id="automoveForward"
        ></button>
    </section>
</template>

<style scoped>
    #viewer {
        width: 100vw;
        height: 100vh;
    }

    #additionalNavigation {
        position: fixed;
        top: 15px;
        left: calc(50vw - 60px);
        width: 120px;
        height: 40px;
        box-shadow: 0 0 10px 3px rgb(0, 0, 0, 0.3);
        display: flex;
        background-color: white;
        border-radius: 20px;
    }

    button {
        width: 40px;
        height: 40px;
        border-radius: 20px;
        border: none;
        background-color: white;
        color: rgb(32, 32, 32);
        display: flex;
        justify-content: center;
        align-items: center;
    }

    button:disabled {
        background-color: rgb(235, 235, 235) !important;
        color: rgb(32, 32, 32) !important;
        cursor: not-allowed !important;

        img {
            filter: brightness(500%) !important;
        }
    }

    button:hover {
        background-color: red;
        color: white;
        cursor: pointer;

        img {
            filter: brightness(10000%);
        }
    }

    .selected {
        background-color: red;
        color: white;
        cursor: default !important;

        img {
            filter: brightness(10000%);
        }
    }
    
    img {
        width: 20px;
        height: 20px;
    }

    #automoveForward {
        rotate: 180deg;
    }

    .darkened {
        background-color: rgb(235, 235, 235) !important;
        filter: brightness(0.85);
    }

    @media (orientation: portrait) {
        #viewer {
            height: 60vh;
        }
    }
</style>