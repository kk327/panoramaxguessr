<script setup>
    import { useTemplateRef, onUnmounted, watch, ref } from 'vue';

    const props = defineProps([
        "visibleInfo"
    ]);

    const emit = defineEmits([
        "close"
    ]);

    const focusable = [useTemplateRef("article"), useTemplateRef("button")];
    const settings = ref([
        {"name": "startOn360"}, 
        {"name": "neverStartOnOne"}, 
        {"name": "checkerboardMarker"}
    ]);

    for (let setting of settings.value) {
        if (localStorage.getItem(setting.name)) {
            setting.enabled = true;
        } else {
            setting.enabled = false;
        }
    }

    const focusListener = (e) => {
        if (!focusable.includes(e.target.value)) {
            focusable[0].value.focus();
        }
    }  

    addEventListener("focus", focusListener)

    onUnmounted(() => {
        removeEventListener("focus", focusListener);
    })

    watch(settings, () => {
        for (let setting of settings.value) {
            if (setting.enabled && !localStorage.getItem(setting.name)) {
                localStorage.setItem(setting.name, true);
            } else if (!setting.enabled && localStorage.getItem(setting.name)) {
                localStorage.removeItem(setting.name);
            }
        }
    }, { deep: true })
</script>

<template>
    <main>
        <article 
            v-if="visibleInfo == 'about'"
            ref="article"
            tabindex="0"
        >
            <h1>PanoramaxGuessr against Google Street View</h1>
            <p>
                PanoramaxGuessr is a FOSS alternative to GeoGuessr and other similar games.
                The main difference is that it uses Panoramax instead of Google Street View.
                Not wanting to use applications that are based on Google Street View isn't just about it being owned by Google, but also about the fact that everytime you use them Google gets money from that. 
                Using Google Street View in an application not made by Google costs the people that run it money, everytime someone views a photo in it Google requires them to pay a bit.
                That's why playing the original GeoGuessr requires a monthly subscription, and free to play alternatives that use Google Street View usually ask for donations.
                That also means that everytime you play GeoGuessr or an alternative that also uses Google Street View, Google gets money. Just from you playing it.
                And so, if you're into FOSS and dislike big tech including Google but like GeoGuessr, you're one of the people that I made this for.
                This project can also be treated as an attempt to make an unexpensive to run GeoGuessr like game, but being against big tech was the main purpose.
            </p>
            <h1>Gameplay differences</h1>
            <p>
                Panoramax works quite a bit differently from Google Street View, yet still similarly. 
                The first thing that you might notice, depending on your luck, is that only some of the photos are 360°.
                The reason for that is that the photos are taken by regular people, and not everyone is going to buy a 360° camera just to take photos for a Google Street View alternative.
                Although that likely seems like a limitation at first, it can be taken as an additional challenge, like a feature. 
                If you start in a city in a location with 2D photo coverage only, you can try to find a 360° sequence to start being able to see more information.
                Although if you really dislike 2D photos, there's a setting that allows you to always start on a 360° sequence.
                I just mentioned sequences, those are the second difference. 
                Imagine a person moving from one place to another one, taking a photo every a moment. Then they upload them to Panoramax, and that's a sequence.
                When you use the white arrow buttons to move, you're moving along the sequence that you're currently on. 
                If you see an orange arrow button, it means that there's another sequence nearby and you can switch to it by clicking it.
                You can also navigate through a sequence by automoving, as the name suggests, it moves you automatically.
                To enable automoving click on the double arrow buttons at the top of the screen, and to stop automoving click the pause button between them.
                Automoving forward will make you automove in the direction in which the person taking the photos was moving, and automoving backwards will automove you in the opposite direction.
                The last major difference is that there's significantly less photos than on Google Street View, and that they're mostly from France.
            </p>
            <h1>How to play in general</h1>
            <p>
                You're put into a random real life place, and your goal is to find where you are and select the location on the map.
                To do so you move around and try to find information. There's a chance of being in a place with only one photo, in that case you have to place your guess without moving. It's possible to remove that chance with a setting though.
                You can move by a bit by clicking the single arrow buttons, and you can start automoving by clicking the double arrow buttons at the top of the screen.
                There's also a button to go back to the primary location, which is below the map on the screen. Left to it is a compass that you can use to your advantage as well.
                When you're finished looking for where you are, select your guess on the map and click the Guess button. 
                You'll see a fullscreen map with the correct location marked with a green marker on it.
                In addition to navigating using clicking, there are keyboard bindings. 9 - move forward in the direction in which the person taking the photos was moving in, 3 - move backwards, space - start/stop automoving forward, b - automove backwards, arrow keys - rotate.
            </p>
            <p>
                <a href="https://codeberg.org/k327/panoramaxguessr">Codeberg</a> 
                | 
                <a href="https://github.com/kk327/panoramaxguessr">Github</a>
            </p>
        </article>

        <article
            v-else
            ref="article"
            tabindex="0"
        >
            <h1 id="settingsHeader">Settings</h1>
            <section id="settings">
                <label>
                    <input 
                        type="checkbox"
                        v-model="settings[0].enabled"
                    >
                    Always start on a 360° sequence
                </label>
                <label>
                    <input 
                        type="checkbox"
                        v-model="settings[1].enabled"
                    >
                    Never start on a sequence with just one photo
                </label>
                <label>
                    <input 
                        type="checkbox"
                        v-model="settings[2].enabled"
                    >
                    Checkerboard styled correct location marker
                </label>
            </section>
            <p>When you change a setting, it will be stored in localStorage.</p>
        </article>

        <button 
            @click="$emit('close')"
            ref="button"
        >
            <img 
                src="@/assets/close.png"
                alt="X"
            >
        </button>
    </main>
</template>

<style scoped>
    main {
        position: fixed;
        background-color: rgb(0, 0, 0, 0.925);
        width: 100vw;
        height: 100vh;
        left: 0;
        top: 0;
        color: white;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    button {
        position: fixed;
        top: 15px;
        right: 15px;
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

    button:hover {
        background-color: red;
        color: white;
        cursor: pointer;

        img {
            filter: brightness(10000%);
        }
    }

    img {
        width: 20px;
        height: 20px;
    }

    article {
        overflow-y: auto;
        width: 1000px;
        max-height: calc(100vh - 140px);
        max-width: calc(100vw - 60px);
        text-align: center;
    }

    h1 {
        margin: 0;
        padding-top: 10px;
    }

    a {
        color: red;
        text-decoration: none;
    }

    #settings {
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    label {
        margin: 8px;
        cursor: pointer;
    }

    input {
        cursor: pointer;
        accent-color: red;
    }

    label:has(input:checked) {
        font-weight: 600;
    }

    #settingsHeader {
        margin: 7.5px;
    }
</style>