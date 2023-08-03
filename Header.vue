<template>
    <section class="top-bar">
        <div class="left-side">
            <div class="branding">
                <router-link to="/">
                    <img
                        v-if="
                            branding && branding.image && branding.image != null
                        "
                        :src="branding ? branding.image : ''"
                        alt=""
                    />
                    <img v-else :src="app_url + '/img/logo.png'" alt="" />
                </router-link>
            </div>

            <h4 v-if="currentRouteName == 'home'" class="page-title">
                Dashboard
            </h4>
            <router-link
                v-if="
                    currentRouteName === 'Connections' ||
                        currentRouteName === 'UserProfile' ||
                        currentRouteName === 'Matches' ||
                        currentRouteName === 'Groups'
                "
                class="back-nav raw-flex align-center hide-mobile"
                :to="currentRouteName == 'UserProfile' ? '/connections' : '/'"
            >
                <i class="icon-arrow-right-round"></i>
                <span>Back</span>
            </router-link>
            <router-link
                v-if="currentRouteName === 'Wall'"
                class="back-nav hide-mobile"
                to="/group"
            >
                <i class="icon-arrow-right-round"></i>
                <span>Back</span>
            </router-link>
        </div>
        <div class="center-main-nav">
            <router-link
                v-if="canAccess('Connections') && $helpers.excludeContactRoles(rolesCount, userRoles, 'Connections')"
                to="/connections"
                class="nav-link"
            >
                <i class="icon-connections"></i
                ><span>{{ connectionsLabel }}s</span>
            </router-link>

            <router-link
                v-if="canAccess('Groups') && $helpers.excludeContactRoles(rolesCount, userRoles, 'Groups')"
                to="/group"
                class="nav-link"
            >
                <i class="icon-groups"></i><span>{{ groupsLabel }}s</span>
            </router-link>

            <a
                v-if="isResourcesAccess && canAccess('Resources') && $helpers.excludeContactRoles(rolesCount, userRoles, 'Resources')"
                :href="webBaseUrl + '/resources'"
                class="nav-link"
            >
                <i class="icon-resources"></i>
                <span>{{ resourcesLabel }}s</span>
            </a>
        </div>
        <div class="right-side">
            <div class="bell-holder">
                <router-link to="/user/messenger" class="messenger-link"  v-if="canAccess('Messenger') || (canAccess('Groups') && messengerGroupsCount > 0 ) || (connectedAdminsCount > 0)">
                    <i class="icon-messenger">
                        <span
                            v-if="this.$store.getters.unreadCount > 0"
                            class="update-dot"
                        ><i>{{ this.$store.getters.unreadCount }}</i></span>
                    </i>
                </router-link>
                <a href="javascript:;" class="notification-toggle">
                    <i class="icon-bell">
                        <span
                            v-if="notifications.length > 0"
                            class="update-dot"
                        ><i>{{ notifications.length }}</i></span>
                    </i>
                </a>
            </div>
            <div class="profile-widget">
                <a
                    href="#"
                    class="profile-name"
                    @click.prevent="profileWidget()"
                >
                    <img v-if="user" :src="user ? user.headshot : ''" alt="" />
                    <span v-if="user"
                        >{{ user.first_name }} {{ user.last_name }}</span
                    >
                    <i class="icon-arrow-down"></i>
                </a>
                <div class="widget-container">
                    <router-link
                        to="/profile"
                        class="link-secondary font-size-1r mar-b-5"
                        >My Profile</router-link>
                    <a v-if="existing_user_id > 0"
                        class="link-secondary font-size-1r mar-b-5"
                        href="/switch_back"
                    >
                        Switch Back
                    </a>
                    <a
                        class="logout-action hide-desktop blocked"
                        :href="logOutUrl"
                        @click.prevent="logout"
                    >
                        <i class="icon-logout"></i>
                        <span>Logout</span>
                    </a>
                    <div
                        class="control-panel-wrapper"
                        v-if="companiesData.length > 1"
                    >
                        <div class="select-company-wrapper mar-b-10">
                            <p class="mar-b-5">Select Company</p>
                            <div class="custom-select">
                                <span class="custom-selected"
                                    >Select Company</span
                                >
                                <ul class="custom-options">
                                    <li
                                        v-for="(company,
                                        index) in companiesData"
                                        :key="index"
                                        class="c-option userCompanies"
                                        :class="{
                                            'c-selected':
                                                user != null &&
                                                user.referral_id ==
                                                    company.referral_id
                                        }"
                                    >
                                        <a
                                            href="#"
                                            :data-id="company.id"
                                            @click.prevent="changeCompayUser"
                                        >
                                            {{ company.company_name }}
                                        </a>
                                    </li>
                                </ul>
                            </div>
                        </div>
                        <!--                        <div class = "role-toggle-wrapper">
                            <p class = "mar-b-5">Choose active Role</p>
                            <div class = "role-toggle-container">
                                <input name = "role" type = "radio" href = "#" v-for = "(userRole, index1) in rolesData"
                                       :key = "index1"
                                       :class = "[(userRole.role_id == 3 ? 'champion' : 'member'), (rolesData.length == 1 ? 'cursor-normal' : '')]"
                                       :data-id = "userRole.id" @change = "changeCompayUser($event)"
                                       :checked = "user != null && userRole.id == user.id" />
                            </div>
                        </div>-->
                    </div>
                </div>
            </div>
            <div class="logout">
                <a
                    class="logout-action"
                    :href="logOutUrl"
                    @click.prevent="logout"
                >
                    <i class="icon-logout"></i>
                </a>
            </div>
        </div>
    </section>
</template>

<script>
import { mapActions, mapGetters } from "vuex";
import axios from "../../store/axios";
import store from "../../store";
import switchUserService from "../../services/switchUser";
import to from "await-to-js";
import { updateCustomSelect } from "../../assets/js/lib";

export default {
    name: "header-bar",
    data() {
        return {
            app_url: app_url,
            existing_user_id: existing_user_id,
            logOutUrl: app_url + "/logout",
            prevRoute: null,
            csrf: csrf_token,
            webBaseUrl: window.location.origin
        };
    },
    computed: {
        ...mapGetters([
            "user",
            "companiesData",
            "rolesData",
            "branding",
            "isAuthenticated",
            "authUserId",
            "isResourcesAccess",
            "notifications",
            "rolesCount",
            "userPermissions",
            "userRoles",
            "connectionsLabel",
            "groupsLabel",
            "resourcesLabel",
            "unreadCount",
            "connectedAdminsCount",
            "messengerGroupsCount"
        ]),
        user() {
            return this.$store.getters.user;
        },
        branding() {
            //if ( this.$store.getters.branding.image == null || this.$store.getters.branding.image == '' ) this.$store.getters.branding.image = this.app_url + '/img/logo.png';
            return this.$store.getters.branding;
        },
        isResourcesAccess() {
            return this.$store.getters.isResourcesAccess;
        },
        currentRouteName() {
            return this.$route.name;
        },
        previousRoute() {
            return this.prevRoute;
        }
    },
    created: function() {
        if (!(this.companiesData && this.companiesData.length)) {
            this.getUserRolesCompanies();
        }
        this.getUnreadMessagesCount();
        this.hasAdminConnections();

    },
    mounted: function () {
        window.addEventListener('load', () => {
            // run after everything is in-place
            this.$nextTick(function () {
                this.initSocketEvent()
            })
        });
    },
    watch: {
        authUserId: function() {
            this.getUserRolesCompanies();
        },
    },
    methods: {
        ...mapActions(["changeUserRole"]),

        logout() {
            this.$store.dispatch("logout");
        },
        redirectToGroups() {
            window.location.href = "/groups";
        },
        redirectToResources() {
            window.location.href = "/resources";
        },
        async initSocketEvent() {
            await this.sleep(5000);
            this.$socket.on('new_message',async data => {
                this.$store.commit("unreadCount", this.$store.getters.unreadCount+1);
            })
        },
        sleep(ms) {
          return new Promise((resolve) => {
            setTimeout(resolve, ms);
          });
        },
        profileWidget() {
            console.log(this.$route.name);
            let dis =
                $(event.target).closest(".profile-name") || $(event.target);
            let widget = dis.siblings(".widget-container");
            if (!widget.hasClass("open-widget")) {
                widget.addClass("open-widget");
            } else {
                widget.removeClass("open-widget");
            }
        },
        getUnreadMessagesCount() {
            axios.get ( 'messengers/conversations/unread/count' ).then (async response => {
                this.$store.commit("unreadCount", Number(response.data.data));
                console.log(response.data.data)
            } ).catch ( err => {
                console.log ( err )
            } );
        },
        getUserRolesCompanies() {
            axios
                .post("user/companies-roles")
                .then(res => {
                    let response = res.data;
                    if (!(response.status == 200)) return;
                    this.$store.commit("userCompaniesData", {
                        companies_data: response.data.companies
                        //roles_data : response.data.roles
                    });
                    Vue.nextTick(function() {
                        updateCustomSelect();
                    });
                })
                .catch(err => {
                    console.log(err);
                });
        },
        async changeCompayUser(event) {
            let userId = event.target.dataset.id;
            if (userId != this.user.id) {
                this.showLoader();
                /* Timezone is required for authentication token expiration */
                let timezone_offset_minutes = new Date().getTimezoneOffset();
                timezone_offset_minutes =
                    timezone_offset_minutes == 0 ? 0 : -timezone_offset_minutes;
                let data = {
                    time_zone: timezone_offset_minutes,
                    user_id: userId
                    //'access_token' : localStorage.getItem ( "token" )
                };
                const [err, res] = await to(
                    switchUserService.changeUserRole(data)
                );
                if (err) {
                    this.hideLoader();
                    this.error("Somethign went wrong. Please try again later.");
                } else {
                    this.hideLoader();
                    //localStorage.removeItem ( "expirationDate" );
                    //localStorage.removeItem ( "token" );
                    localStorage.removeItem("userId");
                    localStorage.removeItem("user");
                    localStorage.removeItem("headerInformation");
                    localStorage.removeItem("checklists");
                    window.location = window.location.origin;
                }
            }
        },
        canAccess(module) {
            return this.$helpers.canAccess(this.userPermissions, module);
        },
        hasAdminConnections() {
            axios.get ( 'messengers/get-admin-connections-count-web' ).then ( response => {
                let res = response.data.data;
                this.$store.commit("connectedAdminsCount", parseInt(res.connected_admins_count));
                this.$store.commit("messengerGroupsCount", parseInt(res.groups_count));
            } ).catch ( err => {
                console.log ( err )
            } );
        }
    },
    beforeRouteEnter(to, form, next) {
        next(vm => {
            console.log(form.path);
            vm.prevRoute = form.path;
        });
    }
};
</script>

<style lang="scss" scoped>
@import "resources/assets/sass/abstracts/_variables.scss";

.top-bar {
    background: color(white);
    display: flex;
    justify-content: space-between;
    border-bottom: 1px solid #e0e0e0;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    z-index: 999;

    .left-side {
        display: flex;
        align-items: center;

        .back-nav {
            margin-left: 20px;
        }

        .branding {
            display: flex;
            background: color(primary);
            padding: 11px 10px;

            a {
                display: flex;
                align-items: center;
                justify-content: center;
            }

            img {
                max-width: 100px;
                height: auto;
                max-height: 40px;
                object-fit: cover;

                @include mobile {
                    max-width: 80px;
                }
            }
        }

        .page-title {
            display: none;
            margin-left: 15px;
            margin-bottom: 0;

            @include desktop {
                display: block;
                font-family: $bold;
                font-size: 23px;
            }
        }
    }

    .right-side {
        display: flex;
        align-items: center;
        padding-right: 20px;

        .bell-holder {
            display: flex;
            align-items: center;

            a {
                font-size: 20px;
                color: color(secondary);

                &.messenger-link {
                    font-size: 17px;
                    margin-right: 15px;
                }

                .icon-bell,
                .icon-messenger {
                    position: relative;

                    .update-dot {
                            display: inline-block;
                            position: absolute;
                            left: calc(100% - 5px);
                            bottom: calc(100% - 5px);
                            color: white;
                            font-size: 11px;
                            text-align: center;

                        &:before {
                            content: "";
                            position: absolute;
                            min-width: 15px;
                            top: 50%;
                            left: 50%;
                            transform: translate(-50%, -50%);
                            aspect-ratio: 1;
                            width: calc(100% + 7px);
                            background: #eb5757;
                            border-radius: 50%;
                        }

                        i {
                            position: relative;
                            font-style: normal;
                            word-break: keep-all;
                        }
                    }
                }
            }
        }

        .profile-widget {
            font-size: 12px;
            display: flex;
            align-items: center;
            margin-left: 20px;
            position: relative;

            img {
                height: 40px;
                width: 40px;
                min-width: 40px;
                object-fit: cover;
                border-radius: 50%;
            }

            .logout-action.hide-desktop {
                @media (max-width: 920px) {
                    display: block;
                }
            }

            .profile-name {
                display: flex;
                align-items: center;
                margin-left: 10px;
                color: color(lighter);

                span {
                    margin-left: 8px;
                }

                .icon-arrow-down {
                    font-size: 8px;
                    margin-left: 5px;

                    @include desktop {
                        margin-right: 10px;
                    }
                }
            }

            .logout-action.hide-desktop {
                color: color(secondary);
                font-size: 1rem;

                span {
                    font-family: $bold;
                }

                i {
                    position: relative;
                    top: 2px;
                }
            }
        }

        .logout {
            display: none;

            @include desktop {
                display: flex;
                border-left: 1px solid #e0e0e0;
                padding-left: 20px;
                height: 100%;
                align-items: center;
            }

            @media (max-width: 920px) {
                display: none;
            }

            .logout-action {
                font-size: 12px;
            }
        }
    }

    .widget-container {
        opacity: 0;
        visibility: hidden;
        position: absolute;
        max-height: 0;
        right: 0;
        top: calc(100% + 7px);
        background: color(white);
        padding: 0;
        width: 100%;
        max-width: max-content;
        border-radius: 6px;
        box-shadow: $dropDown-shadow;
        transition: max-width ease 0.275s, opacity ease 0.275s,
            visibility ease 0.275s, max-height ease 0.275s, padding ease 0.275s;

        &.open-widget {
            opacity: 1;
            visibility: visible;
            max-height: 600px;
            padding: 20px;
        }
    }
}

.center-main-nav {
    display: none;
    align-items: center;

    @media (min-width: 921px) {
        display: flex;
    }
}
</style>
