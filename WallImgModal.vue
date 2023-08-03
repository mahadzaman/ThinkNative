<template>
    <div v-if="imgPopupOpen === true" id="img-viewer" class="popup-wrapper">
        <div @click="$emit('click')" class="popup-backdrop"></div>
        <div class="img-container">
            <img :src="srcLink" alt="">
            <a @click="$emit('click')" href="javascript:void(0)" class="close-img-popup">
                <i class="icon-close"></i>
            </a>
        </div>
    </div>
</template>

<script>

export default {
    name: 'ImageViewer',
    // emits: ['closePopup'],
    props: [
        'imgPopupOpen', 'srcLink'
    ]
}

</script>

<style lang="scss">
    @import "resources/assets/sass/abstracts/_variables.scss";

    .popup-wrapper {
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        z-index: 1024;
        display: flex;
        isolation: isolate;
        align-items: center;

        .popup-backdrop {
            background: rgba(0, 0, 0, 0.7);
            position: absolute;
            width: 100%;
            height: 100%;
            z-index: -1;
        }

        .close-img-popup {
            color: color(white);
            position: absolute;
            left: calc(100% + 10px);
            display: flex;
            bottom: calc(100% + 10px);
        }

        .img-container {
                max-width: calc(100% - 72px);
                max-height: calc(100% - 72px);
                margin: auto;
                display: flex;
                align-items: center;
                justify-content: center;
                position: relative;
                filter: drop-shadow(2px 4px 6px rgba(0, 0, 0, 0.4));

            img {
                max-height: calc(100vh - 72px);
            }
        }
    }
</style>