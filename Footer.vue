<template>
    <footer id = "footer" v-bind:class = "{ footerW: $route.path == '/' }">
        <p class = "copyright">
            &copy; 2022 Think Native. Inc.
        </p>
        <p>
            Need Support? Fill out a support ticket!
            <a target="_blank" href = "https://thinknative.com/support" class = "bordered-btn">Help</a>
        </p>
    </footer>
</template>

<script>
    export default {
        name : "footer-bar"
    };
</script>

<style scoped lang="scss">
    @import "resources/assets/sass/abstracts/_variables.scss";

    .footerW {
        max-width: 1160px;
    }
    footer {
        border-top: 1px solid #d0d7de;
        margin: 30px 15px 0;
        padding: 20px 0;
        text-align: center;

        p {
            font-size: 12px;
            color: color(lighter);

            &:not(.copy-right) {
                margin-top: 5px;
            }
        }

        main + & {
        margin: 50px 20px 0;
            @include desktop {
                margin: 50px auto 0;
            }
        }

        .bordered-btn {
            margin-left: 5px;
        }

        @include desktop {
            display: flex;
            justify-content: space-between;
            flex-wrap: wrap;
            max-width: 1081px;
            margin: 50px 30px 0;
            padding: 20px 30px;
        }

        @media (min-width: 1141px) {
            margin: 50px auto 0;
        }
    }
</style>
