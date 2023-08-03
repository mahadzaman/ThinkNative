<template>
    <main>
        <div class="container show-only-mobile">
            <router-link to="/" class="back-nav blocked">
                <i class="icon-arrow-right-round"></i>
                <span>Back</span>
            </router-link>
        </div>

        <div class="container">
            <section class="page-title-container raw-flex space-between align-center">
                <h2 class="page-title">{{groupsLabel}}s</h2>
            </section>

            <section v-if="groupsDescription" class="groups-description">
                <article v-html="groupsDescription"></article>
            </section>

            <section class="raw-flex">
                <div class="search flex-col-33">
                    <input type="text" class="searchTerm" placeholder="Keyword Search" v-model="searchString" @keydown.enter='getGroupsListing' @keydown.188='getGroupsListing'>
                    <button type="submit" @click="getGroupsListing" class="searchButton btn-de-default">
                        <i class="icon-search colorSearch"></i>
                    </button>
                </div>
            </section>

            <section class="pad-t-30">
                <div class="tabs-header-wrapper">
                    <nav>
                        <div class="tabs-header flex-box align-center space-between nav-tabs nav" role="tablist">
                            <a class="tab-link nav-link active" data-toggle="tab"  id="about-nav" href="#enrolledGroups" role="tab" aria-controls="enrolledGroups" v-on:click="switchGroups('enrolled')" aria-selected="true">
                                <span>Enrolled</span>
                                <span class="bolder">Enrolled</span>
                            </a>
                            <a class="tab-link nav-link " data-toggle="tab" id="discussion-nav" href="#suggestedGroups" role="tab" aria-controls="suggestedGroups" v-on:click="switchGroups('suggested')" aria-selected="true">
                                <span>Suggested</span>
                                <span class="bolder">Suggested</span>
                            </a>
                            <a class="tab-link nav-link" data-toggle="tab"  id="event-nav" href="#allAvailable" role="tab" aria-controls="allAvailable" v-on:click="switchGroups('available')" aria-selected="true">
                                <span>All Available</span>
                                <span class="bolder">All Available</span>
                            </a>
                        </div>
                    </nav>
                </div>

                <div class="tab-content">
                    <div class = "tab-pane fade show active" id = "enrolledGroups" role = "tabpanel"
                            aria-labelledby = "enrolledGroups">
                        <template v-if="Object.keys(enrolledGroups).length > 0">
                            <div class="groups-container" v-for="(enrolledGroup, category) in enrolledGroups" :key="category">
                                <h6 class="category-title">{{ category }}</h6>
                                <div class="groups-row">
                                    <div class="group-tile-col" v-for="(group, index) in enrolledGroup" :key="index">
                                        <router-link :to="groupWallLink(group.unique_id)" class="group-tile-wrapper">
                                            <div class="tile-cover">
                                                <img :src="group.image ? isVideo(group.image, group.icon) : '/img/Default_Groups_Tile_Background.jpg'"
                                                        :style="{top: group.cover_position, transform: 'translate(-50%, 0)'}"
                                                >
                                            </div>
                                            <div class="tile-identity">
                                                <div class="img-col">
                                                    <img :src="group.icon ? group.icon : '/img/Default_Groups_Tile_Icon.png'">
                                                </div>
                                                <div class="info-col">
                                                    <h6 class="group-name">{{ group.name ? group.name : '' }}</h6>
                                                    <div class="tags-container" v-if="group.tags && group.tags.length > 0">
                                                        <span class="tag" v-for="(tag, tagIndex) in group.tags.slice(0, 4)" :key="tagIndex">{{ tag.name }}</span>
                                                    </div>
                                                </div>
                                                <p class="about-text tooltip-c" tooltip-position="top" v-if="group.description && group.description.length < 58">
                                                    {{ group.description }}
                                                    <span class="tooltip-text">
                                                        <span class="line-clamp-10">{{ group.description }}</span>
                                                    </span>
                                                </p>
                                                <p class="about-text tooltip-c" tooltip-position="top" v-else>
                                                    {{ group.description ? group.description.substring(0, 58)+"..." : "" }}
                                                    <span class="tooltip-text">
                                                        <span class="line-clamp-10">{{ group.description }}</span>
                                                    </span>
                                                </p>
                                            </div>
                                            <div class="mar-t-auto">
                                                <p class="group-capacity" v-if="(selectedTab == 'enrolled' && group.is_capacity == 1 && group.member.length == 0) && !isGroupFull(group)">{{ getCapacityLabelText(group) }}</p>
                                                <div class="tile-footer">
                                                    <!-- If the group has at least one group member other than group owner -->
                                                    <div class="mem-pics-container" v-if="group.enrolled_users && group.enrolled_users.length > 0">
                                                        <img v-for="(user, userIndex) in group.enrolled_users.slice(0, 3)" :key="userIndex" :src="user.headshot ? user.headshot : '/img/default.jpg'">
                                                        <img v-if="group.user && group.enrolled_users && group.enrolled_users.length < 3" :src="group.user.headshot ? group.user.headshot : '/img/default.jpg'">
                                                        <span class="enroll-count">{{ group.enrolled_users.length + 1 }} Enrolled</span>
                                                    </div>
                                                    <!-- If the group has group owner as group member only -->
                                                    <div class="mem-pics-container" v-if="group.enrolled_users && group.enrolled_users.length == 0">
                                                        <img v-if="group.user" :src="group.user.headshot ? group.user.headshot : '/img/default.jpg'">
                                                        <span class="enroll-count">{{ 1 }} Enrolled</span>
                                                    </div>
                                                    <a v-if="selectedTab == 'enrolled' && group.requests.length > 0 && group.requests[0].is_invite == 1" @click.prevent="acceptGroupRequest(group.id)" href="javascript:void(0)" class="secondary-btn mar-l-auto">Accept</a>
                                                    <a v-else-if="selectedTab == 'enrolled' && group.requests.length > 0 && group.requests[0].is_invite == 0" href="javascript:void(0)" class="secondary-btn mar-l-auto">Pending</a>
                                                    <a v-else-if="isGroupFull(group)" class="less-pad-btn disabled-btn mar-l-auto">{{groupsLabel}} Full</a>
                                                    <a v-else-if="selectedTab == 'enrolled' && group.private == 0 && group.member.length == 0" @click.prevent="joinGroupRequest(group.id)" href="javascript:void(0)" class="secondary-btn mar-l-auto">Request</a>
                                                    <a v-else-if="selectedTab == 'enrolled' && group.private == 2 && group.member.length == 0" @click.prevent="joinGroupRequest(group.id)" href="javascript:void(0)" class="secondary-btn mar-l-auto">Join</a>
                                                    <a v-else class="secondary-btn mar-l-auto border-btn" @click.prevent="leaveGroup(group.id)" href="javascript:void(0)">Leave</a>
                                                </div>
                                            </div>
                                        </router-link>
                                    </div>
                                </div>
                            </div>
                        </template>

                        <template v-else-if="!isShowLoader">
                            <p>You have not Enrolled in any {{ groupsLabel }}. Check out <b class="underline">Suggested</b> tab</p>
                        </template>
                    </div>

                    <div class = "tab-pane fade" id = "suggestedGroups" role = "tabpanel"
                            aria-labelledby = "suggestedGroups">
                        <template v-if="Object.entries(suggestedGroups).length > 0">
                            <div class="groups-container" v-for="(suggestedGroup, category) in suggestedGroups" :key="category">
                                <h6 class="category-title">{{ category }}</h6>
                                <div class="groups-row">
                                    <div class="group-tile-col" v-for="(group, index) in suggestedGroup" :key="index">
                                        <router-link :to="groupWallLink(group.unique_id)" class="group-tile-wrapper">
                                            <div class="tile-cover">
                                                <img :src="group.image ? isVideo(group.image, group.icon) : '/img/Default_Groups_Tile_Background.jpg'"
                                                        :style="{top: group.cover_position, transform: 'translate(-50%, 0)'}"
                                                >
                                            </div>
                                            <div class="tile-identity">
                                                <div class="img-col">
                                                    <img :src="group.icon ? group.icon : '/img/Default_Groups_Tile_Icon.png'">
                                                </div>
                                                <div class="info-col">
                                                    <h6 class="group-name">{{ group.name ? group.name : '' }}</h6>
                                                    <div class="tags-container" v-if="group.tags && group.tags.length > 0">
                                                        <span class="tag" v-for="(tag, tagIndex) in group.tags.slice(0, 4)" :key="tagIndex">{{ tag.name }}</span>
                                                    </div>
                                                </div>
                                                <p class="about-text tooltip-c" tooltip-position="top" v-if="group.description && group.description.length < 58">
                                                    {{ group.description }}
                                                    <span class="tooltip-text">
                                                        <span class="line-clamp-10">{{ group.description }}</span>
                                                    </span>
                                                </p>
                                                <p class="about-text tooltip-c" tooltip-position="top" v-else>
                                                    {{ group.description ? group.description.substring(0, 58)+"..." : "" }}
                                                    <span class="tooltip-text">
                                                        <span class="line-clamp-10">{{ group.description }}</span>
                                                    </span>
                                                </p>
                                            </div>
                                            <div class="mar-t-auto">
                                                <p class="group-capacity" v-if="(selectedTab == 'enrolled' && group.is_capacity == 1 && group.member.length == 0) && !isGroupFull(group)">{{ getCapacityLabelText(group) }}</p>
                                                <div class="tile-footer">
                                                    <!-- If the group has at least one group member other than group owner -->
                                                    <div class="mem-pics-container" v-if="group.enrolled_users && group.enrolled_users.length > 0">
                                                        <img v-for="(user, userIndex) in group.enrolled_users.slice(0, 3)" :key="userIndex" :src="user.headshot ? user.headshot : '/img/default.jpg'">
                                                        <img v-if="group.user && group.enrolled_users && group.enrolled_users.length < 3" :src="group.user.headshot ? group.user.headshot : '/img/default.jpg'">
                                                        <span class="enroll-count">{{ group.enrolled_users.length + 1 }} Enrolled</span>
                                                    </div>
                                                    <!-- If the group has group owner as group member only -->
                                                    <div class="mem-pics-container" v-if="group.enrolled_users && group.enrolled_users.length == 0">
                                                        <img v-if="group.user" :src="group.user.headshot ? group.user.headshot : '/img/default.jpg'">
                                                        <span class="enroll-count">{{ 1 }} Enrolled</span>
                                                    </div>
                                                    <a v-if="selectedTab == 'suggested' && group.requests.length > 0 && group.requests[0].is_invite == 1" @click.prevent="acceptGroupRequest(group.id)" href="javascript:void(0)" class="secondary-btn mar-l-auto">Accept</a>
                                                    <a v-else-if="selectedTab == 'suggested' && group.requests.length > 0 && group.requests[0].is_invite == 0" href="javascript:void(0)" class="secondary-btn mar-l-auto">Pending</a>
                                                    <a v-else-if="isGroupFull(group)" class="less-pad-btn disabled-btn mar-l-auto">{{groupsLabel}} Full</a>
                                                    <a v-else-if="selectedTab == 'suggested' && group.private == 0 && group.member.length == 0" @click.prevent="joinGroupRequest(group.id)" href="javascript:void(0)" class="secondary-btn mar-l-auto">Request</a>
                                                    <a v-else-if="selectedTab == 'suggested' && group.private == 2 && group.member.length == 0" @click.prevent="joinGroupRequest(group.id)" href="javascript:void(0)" class="secondary-btn mar-l-auto">Join</a>
                                                </div>
                                            </div>
                                        </router-link>
                                    </div>
                                </div>
                            </div>
                        </template>

                        <template v-else-if="!isShowLoader">
                            <p>No suggestions available yet!</p>
                        </template>
                    </div>

                    <div class = "tab-pane fade" id = "allAvailable" role = "tabpanel" aria-labelledby = "allAvailable">
                        <template v-if="Object.keys(availableGroups).length > 0">
                            <div class="groups-container" v-for="(availableGroup, category) in availableGroups" :key="category">
                                <h6 class="category-title">{{ category }}</h6>
                                <div class="groups-row">
                                    <div class="group-tile-col" v-for="(group, index) in availableGroup" :key="index">
                                        <router-link :to="groupWallLink(group.unique_id)" class="group-tile-wrapper">
                                            <div class="tile-cover">
                                                <img :src="group.image ? isVideo(group.image, group.icon) : '/img/Default_Groups_Tile_Background.jpg'"
                                                        :style="{top: group.cover_position, transform: 'translate(-50%, 0)'}"
                                                >
                                            </div>
                                            <div class="tile-identity">
                                                <div class="img-col">
                                                    <img :src="group.icon ? group.icon : '/img/Default_Groups_Tile_Icon.png'">
                                                </div>
                                                <div class="info-col">
                                                    <h6 class="group-name">{{ group.name ? group.name : '' }}</h6>
                                                    <div class="tags-container" v-if="group.tags && group.tags.length > 0">
                                                        <span class="tag" v-for="(tag, tagIndex) in group.tags.slice(0, 4)" :key="tagIndex">{{ tag.name }}</span>
                                                    </div>
                                                </div>
                                                <p class="about-text tooltip-c" tooltip-position="top" v-if="group.description && group.description.length < 58">
                                                    {{ group.description }}
                                                    <span class="tooltip-text">
                                                        <span class="line-clamp-10">{{ group.description }}</span>
                                                    </span>
                                                </p>
                                                <p class="about-text tooltip-c" tooltip-position="top" v-else>
                                                    {{ group.description ? group.description.substring(0, 58)+"..." : "" }}
                                                    <span class="tooltip-text">
                                                        <span class="line-clamp-10">{{ group.description }}</span>
                                                    </span>
                                                </p>
                                            </div>
                                            <div class="mar-t-auto">
                                                <p class="group-capacity" v-if="(selectedTab == 'enrolled' && group.is_capacity == 1 && group.member.length == 0) && !isGroupFull(group)">{{ getCapacityLabelText(group) }}</p>
                                                <div class="tile-footer">
                                                    <!-- If the group has at least one group member other than group owner -->
                                                    <div class="mem-pics-container" v-if="group.enrolled_users && group.enrolled_users.length > 0">
                                                        <img v-for="(user, userIndex) in group.enrolled_users.slice(0, 3)" :key="userIndex" :src="user.headshot ? user.headshot : '/img/default.jpg'">
                                                        <img v-if="group.user && group.enrolled_users && group.enrolled_users.length < 3" :src="group.user.headshot ? group.user.headshot : '/img/default.jpg'">
                                                        <span class="enroll-count">{{ group.enrolled_users.length + 1 }} Enrolled</span>
                                                    </div>
                                                    <!-- If the group has group owner as group member only -->
                                                    <div class="mem-pics-container" v-if="group.enrolled_users && group.enrolled_users.length == 0">
                                                        <img v-if="group.user" :src="group.user.headshot ? group.user.headshot : '/img/default.jpg'">
                                                        <span class="enroll-count">{{ 1 }} Enrolled</span>
                                                    </div>
                                                    <a v-if="selectedTab == 'available' && group.requests.length > 0 && group.requests[0].is_invite == 1" @click.prevent="acceptGroupRequest(group.id)" href="javascript:void(0)" class="secondary-btn mar-l-auto">Accept</a>
                                                    <a v-else-if="selectedTab == 'available' && group.requests.length > 0 && group.requests[0].is_invite == 0" href="javascript:void(0)" class="secondary-btn mar-l-auto">Pending</a>
                                                    <a v-else-if="isGroupFull(group)" class="less-pad-btn disabled-btn mar-l-auto">{{groupsLabel}} Full</a>
                                                    <a v-else-if="selectedTab == 'available' && group.private == 0 && group.member.length == 0" @click.prevent="joinGroupRequest(group.id)" href="javascript:void(0)" class="secondary-btn mar-l-auto">Request</a>
                                                    <a v-else-if="selectedTab == 'available' && group.private == 2 && group.member.length == 0" @click.prevent="joinGroupRequest(group.id)" href="javascript:void(0)" class="secondary-btn mar-l-auto">Join</a>
                                                </div>
                                            </div>
                                        </router-link>
                                    </div>
                                </div>
                            </div>
                        </template>

                        <template v-else-if="!isShowLoader">
                            <p>There are no {{groupsLabel}}s available in the Community</p>
                        </template>
                    </div>
                </div>
            </section>
        </div>

        <actions-bar class="hide-desktop" />
    </main>
</template>

<script>
    // import axios from "../store/axios";
    import { mapGetters } from "vuex";
    import to from "await-to-js";
    import service from "../services/groups";
    import ActionsBar from "./layouts/Actions";
    // import { equalHeightCards } from "../assets/js/lib";

    export default {
        name: 'Groups',
        components : { ActionsBar },
        data () {
            return {
                isShowLoader : false,
                selectedTab : 'enrolled',
                searchString : '',
                groupsDescription: ''
            };
        },
        computed: {
            ...mapGetters(["enrolledGroups", "suggestedGroups", "availableGroups", "authUserId", "user", "userPermissions", "groupsLabel"]),
        },
        created: function () {
            this.hasUserGroupsAccess();
            this.getGroupsListing();
        },
        updated: function () {
            /* Vue.nextTick(function () {
                equalHeightCards("group-tile-wrapper");
                $(window).on("resize", function (){
                    equalHeightCards("group-tile-wrapper");
                })
            }); */
        },
        methods: {
            async getGroupsListing (){
                if(!this.isShowLoader){
                    this.isShowLoader = true;
                    console.log(this.availableGroups);
                    this.showLoader();
                }
                const [ err , res ] = await to ( service.fetchGroups({search_string : this.searchString, 'show_type' : this.selectedTab, 'id' : this.authUserId } ) );

                if ( err ) {
                    console.log(err);
                }
                else {
                    const response = res.data.data;
                    this.groupsDescription = response.groups_description;
                    if(this.selectedTab == 'enrolled'){
                        this.$store.commit("enrolledGroups", response.groups);
                        console.log(this.enrolledGroups)
                    }else if(this.selectedTab == 'suggested'){
                        this.$store.commit("suggestedGroups", response.groups);
                        console.log(this.suggestedGroups)

                    }else{
                        this.$store.commit("availableGroups", response.groups);
                        console.log(this.availableGroups)
                    }
                }
                if(this.isShowLoader){
                    this.isShowLoader = false;
                    this.hideLoader();
                }
            },

            switchGroups(type) {
                this.selectedTab = type;
                this.getGroupsListing();
            },

            async acceptGroupRequest(groupId) {
                this.showLoader();
                const [err, res] = await to(service.acceptGroupRequest({ group_id: groupId }));
                if (err) {
                    this.hideLoader();
                    this.error(err.response.data.message);
                }
                else {
                    let result = res.data;
                    this.success(result.message);
                    this.hideLoader();
                    await this.getGroupsListing();
                }
            },

            async joinGroupRequest(groupId) {
                this.showLoader();
                const [err, res] = await to(service.joinGroupRequest({ group_id: groupId }));
                if (err) {
                    this.hideLoader();
                    this.error(err.response.data.message);
                }
                else {
                    this.hideLoader();
                    let result = res.data;
                    this.success(result.message);
                    if (result.data.text == 'Joined') await this.$router.push('group/' + result.data.unique_id + '/wall');//window.location = result.data.ref;
                    else {
                        await this.getGroupsListing();
                    }
                }
            },

            async leaveGroup(groupId) {
                this.$confirm({
                    message: 'Are you sure you want to leave this ' + this.groupsLabel.toLowerCase() +'?',
                    button: {
                        no: "Cancel",
                        yes: "Yes"
                    },
                    /**
                     * Callback Function
                     * @param {Boolean} confirm
                     */
                    callback: async confirm => {
                        if (confirm) {
                            this.showLoader();
                            const [err, res] = await to(service.leaveGroup({ group_id: groupId }));
                            if (err) {
                                this.hideLoader();
                                this.error(err.response.data.message);
                            }
                            else {
                                let result = res.data;
                                this.success(result.message);
                                this.hideLoader();
                                await this.getGroupsListing();
                            }
                        }
                    }
                });
            },

            isVideo(bgImage, iconImage) {
                let basename = bgImage.split(/[\\/]/).pop(),  // extract file name from full path ...
                    // (supports `\\` and `/` separators)
                    pos = basename.lastIndexOf(".");       // get last position of `.`
                if (basename === "" || pos < 1)            // if file name is empty or ...
                    return false;                             //  `.` not found (-1) or comes first (0)

                switch (basename.slice(pos + 1)) {
                    case 'flv':
                    case 'ogg':
                    case 'webm':
                    case 'swf':
                    case 'mpeg':
                    case 'wmv':
                    case 'mov':
                    case 'rm':
                    case 'ram':
                    case 'm4v':
                    case 'avi':
                    case 'mpg':
                    case 'mp4':
                        return '/img/Default_Groups_Tile_Background.jpg';
                }
                return bgImage;
            },
            groupWallLink: function (groupUniqueId) {
                return (this.selectedTab == 'enrolled') ? '/group/' + groupUniqueId + '/wall' : '#';
            },
            getCapacityLabelText: function (groupData) {
                let remainingMember = parseInt(groupData.capacity_value) - parseInt(groupData.enrolled_users.length + 1);
                return remainingMember + ' of ' + groupData.capacity_value + ' Available';
            },
            isGroupFull: function (group) {
                return group.is_capacity == 1 && ((group.enrolled_users.length + 1) >= group.capacity_value) && group.member.length == 0;
            },
            hasUserGroupsAccess: function () {
                if((this.userPermissions) && !this.userPermissions.includes('Groups')) window.location = '/';
            }
        },
        async beforeRouteLeave(tos, from, next) {
            if (tos.params && tos.params.id) {
                this.showLoader();
                const [err, res] = await to(service.getRequest('groups/' + tos.params.id + '/get-about'));
                if (err) {
                }
                else {
                    await this.$store.commit('groupData', res.data.group_data);
                }
                this.hideLoader();
            }
            next();
        }
    }
</script>

<style lang="scss" scoped>
    @import "resources/assets/sass/abstracts/_variables.scss";

    .groups-container {
        background: lighten(color(card-bg), 5%);
        padding: 20px;
        border-radius: $border-radius;
        margin-bottom: 15px;

        .category-title {
            color: color(lighter);
            font-size: 18px;
            font-family: $semiBold;
            font-style: italic;
        }
    }

    .groups-row {
        display: flex;
        flex-wrap: wrap;
        margin-right: -10px;
        margin-left: -10px;
        margin-top: -20px;
    }

    .group-tile-col {
        padding-right: 10px;
        padding-left: 10px;
        flex: 0 1 25%;
        padding-top: 20px;
        min-width: 250px;
        width: 33.33%;

        @include tablet {
            flex: 0 0 50%;
        }

        @include mobile {
            flex: 0 0 100%;
        }
    }

    .group-tile-wrapper {
        background: color(white);
        box-shadow: $card-shadow;
        border-radius: $border-radius;
        height: 100%;
        display: flex;
        flex-direction: column;

        .tile-cover {
            background: #9e9e9e;
            border-top-right-radius: $border-radius;
            border-top-left-radius: $border-radius;
            position: relative;
            overflow: hidden;
            padding-top: 29.4%;

            img {
                width: 100%;
                object-fit: cover;
                border-top-right-radius: $border-radius;
                border-top-left-radius: $border-radius;
                position: absolute;
                left: 50%;
                top: 50%;
                transform: translate(-50%, -50%);
            }
        }

        .tile-identity {
            display: flex;
            flex-wrap: wrap;
            padding-left: 30px;
            padding-right: 8px;
            position: relative;

            .img-col {
                flex: 0 0 50px;

                img {
                    height: 50px;
                    width: 50px;
                    object-fit: cover;
                    border-radius: 50%;
                    box-shadow: 0px 0px 0 2px rgba(255, 255, 255, 0.6);
                    margin-top: -8px;
                    background-color: #f6f6f6;
                }
            }

            .info-col {
                display: flex;
                flex-direction: column;
                flex: 1 1 60%;
                margin-left: 10px;
                padding-top: 8px;

                .group-name {
                    color: color(dark);
                    font-size: 16px;
                    font-family: $bold;
                    display: -webkit-box;
                    -webkit-line-clamp: 2;
                    -webkit-box-orient: vertical;
                    overflow: hidden;
                    text-overflow: ellipsis;
                }

                .tags-container {
                    margin-left: -4px;
                    display: flex;
                    flex-wrap: wrap;

                    .tag {
                        background: color(bg-main);
                        padding: 4px 8px;
                        font-size: 12px;
                        color: color(secondary);
                        border-radius: 20px;
                        margin-bottom: 4px;
                        margin-left: 4px;
                    }
                }
            }

            .about-text {
                font-size: 14px;
                color: color(dark);
                overflow-wrap: anywhere;
                flex: 0 0 100%;
            }
        }

        .group-capacity {
            margin-bottom: 0;
            text-align: right;
            padding-right: 10px;
            font-style: italic;
            font-size: 13px;
        }

        .tile-footer {
            display: flex;
            padding: 10px 8px;
            justify-content: space-between;
            margin-top: auto;

            .mem-pics-container {
                display: flex;
                align-items: center;

                img {
                    position: relative;
                    border-radius: 50%;
                    width: 25px;
                    height: 25px;
                    object-fit: cover;
                    box-shadow: 0px 0px 0 3px rgba(255, 255, 255, 1);

                    &:first-of-type {
                        z-index: 3;
                    }

                    &:nth-of-type(2) {
                        z-index: 2;
                        margin-left: -3px;
                    }

                    &:last-of-type {
                        z-index: 1;
                        margin-left: -3px;
                    }
                }

                .enroll-count {
                    font-size: 12px;
                    color: color(lighter);
                    font-family: $semiBold;
                    margin-left: 8px;
                }
            }
        }
    }

    .tile-grid-column {
        display: grid;
        grid-template-columns: 1fr;
        grid-auto-rows: 1fr;
        grid-gap: 20px;
        margin-bottom: 20px;

        @include tabletZone {
            grid-template-columns: 1fr 1fr 1fr;
            grid-auto-rows: initial;
        }

        @include tablet {
            grid-template-columns: 1fr 1fr;
        }

        @include mobile {
            grid-template-columns: 1fr;
        }
    }

    // Scss for tabs header

    .tabs-header-wrapper {
    margin-top: 30px;
    border-top: $light-card-border;
    width: 100%;
    margin: 0 auto 30px;
    }

    .tabs-header {
        max-width: 528px;
        margin: 0 auto;
        color: color(secondary);
        overflow-x: auto;

        &.flex-box{
            flex-wrap: nowrap;
        }

        &.nav-tabs {
            border-bottom: none;

            .nav-link {
                border: none;
                border-top-left-radius: 0;
                border-top-right-radius: 0;
                &.active,
                &:hover {
                    color: inherit;
                    border: none;
                    background: transparent;
                }
            }
        }

        .tab-link {
            margin-left: 10px;
            position: relative;
            padding: 10px 15px 5px;

            &:after {
                content: "";
                position: absolute;
                top: 0;
                right: 0;
                width: 0;
                height: 1px;
                background: color(secondary);
                transition: width ease 0.2s;
            }

            &.active:after,
            &:hover:after {
                left: 0;
                width: 100%;
            }

            span {
                transition: all ease 0.275s;
            }

            span.bolder {
                font-family: $bold;
                opacity: 0;
                visibility: hidden;
                position: absolute;
                left: 14px;
                bottom: 4px;
            }

            &.active,
            &:hover {
                span.bolder {
                    opacity: 1;
                    visibility: visible;
                }

                span:not(.bolder) {
                    opacity: 0;
                    visibility: hidden;
                }
            }

            &:first-of-type {
                margin-left: 0;
            }
        }
    }

    .mem-pics-container {
        display: flex;
        align-items: center;

        img {
            position: relative;
            border-radius: 50%;
            width: 45px;
            height: 45px;
            object-fit: cover;
            box-shadow: 0px 0px 0 2px rgba(255, 255, 255, 1);

            &:first-of-type {
                z-index: 3;
            }

            &:nth-of-type(2) {
                z-index: 2;
                margin-left: -10px;
            }

            &:last-of-type {
                z-index: 1;
                margin-left: -10px
            }
        }

        .enroll-count {
            font-size: 12px;
            color: color(lighter);
            font-family: $semiBold;
            margin-left: 8px;
        }
    }

    .join-g-btn {
        padding: 10px 37px;
        font-size: 18px;
        font-family: $bold;

        @include mobile {
            margin-top: 10px;
        }
    }

    .white-background-section {
        background: white !important;
    }
</style>
