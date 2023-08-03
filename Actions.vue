<template>
    <section id = "actions-bar" v-if="canAccess('Connections') || canAccess('Groups') || canAccess('Resources')" class="tri-nav-wrapper">
        <div class = "tri-nav">
            <div class = "nav-info-container">
                <h4 class = "card-title">Actions</h4>
            </div>
            <div class = "nav-links">
                <router-link
                    v-if = "canAccess('Connections')"
                    to = "connections" class = "nav-link">
                    <i class = "icon-connections"></i>
                    <span>{{connectionsLabel}}s</span>
                </router-link>
                <router-link
                    v-if="canAccess('Groups')"
                    to = "group" class = "nav-link">
                    <i class = "icon-groups"></i>
                    <span>{{groupsLabel}}s</span>
                </router-link>
                <a @click = "redirectToResources" v-if="canAccess('Resources')" href = "javascript:;" class = "nav-link">
                    <i class = "icon-resources"></i>
                    <span>{{resourcesLabel}}s</span>
                </a>
            </div>
        </div>
    </section>
</template>

<script>
    import { mapGetters } from "vuex";

    export default {
        name : "actions-bar",
        computed : {
            ... mapGetters ( [
                "user",
                "rolesCount",
                "secondaryUser",
                "connectionsLabel",
                "groupsLabel",
                "resourcesLabel",
                "userPermissions"
            ] )
        },
        created : function () {
            //
        },
        methods : {
            redirectToGroups () {
                window.location.href = "/connect/group"
            },
            redirectToResources () {
                window.location.href = '/resources'
            },
            canAccess (module) {
                return this.$helpers.canAccess(this.userPermissions, module);
            }
        }
    };
</script>

<style scoped lang = "scss">
    @import "resources/assets/sass/abstracts/_variables.scss";

    .tri-nav-wrapper,
    .tri-nav-wrapper.hide-desktop {
        @media (max-width: 920px) {
            display: block;
        }
    }

    .tri-nav {
        max-width: 1200px;
        margin: 0 auto;

        @media (min-width: 921px) {
            background: color(card-bg);
            border-radius: $border-radius;
            padding: 30px;
            margin-top: 20px;
        }

        .nav-info-container {
            display: none;

            .card-info {
                color: color(lighter);
                width: 40%;
            }

            @media (min-width: 921px) {
                display: block;
            }
        }

        .nav-links {
            margin: 20px 15px 0;
            padding: 10px 40px;
            display: flex;
            justify-content: space-between;
            border-radius: 32px;
            background: color(white);
            box-shadow: 0 14px 24px rgba(21, 68, 98, 0.1);

            @media (min-width: 921px) {
                background: none;
                box-shadow: none;
                margin-right: -8px;
                margin-left: -8px;
                padding: 0;
            }

            .nav-link {
                display: flex;
                flex-direction: column;
                align-items: center;
                text-align: center;
                color: color(dark);
                padding: 0;

                &.router-link-active {
                    color: color(secondary);
                }

                i {
                    font-size: 26px;

                    @media (min-width: 921px) {
                        background: color(card-bg);
                        border-radius: 50%;
                        padding: 20px;
                        color: color(secondary);
                        margin-bottom: 12px;
                        font-size: 44px;
                    }
                }

                span {
                    font-size: 10px;
                    margin-top: 5px;
                    margin-left: 0;
                    display: inline;

                    @media (min-width: 921px) {
                        font-size: 22px;
                        font-family: $bold;
                    }
                }

                @media (min-width: 921px) {
                    background: color(white);
                    border-radius: $border-radius;
                    padding: 40px;
                    flex: 1 1 33.33%;
                    margin-right: 8px;
                    margin-left: 8px;
                    color: color(newLight);
                }
            }
        }
    }

    @media (min-width: 921px) {
        #actions-bar {
            padding: 0 20px;
            position: relative;
        }
    }

    main {
        .tri-nav {
            transform: translateY(40%);

            .nav-links {
                margin: 20px 0 0;
                padding: 10px 20px;
            }
        }
    }
</style>
