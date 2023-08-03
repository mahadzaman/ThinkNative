<template>
<div>
    <div class="container raw-flex space-between align-center">
        <div v-if="groupData && groupData.is_video_cover == true" class="coverImgBanner">
            <video autoplay loop muted oncontextmenu = "return false;" id = "videoCover" :style="groupCoverPoistion">
                <source :src="groupData.image" />
            </video>
        </div>
        <div  v-if="groupData && groupData.is_video_cover == false" class="coverImgBanner"><img :src="(groupData.image == null) ? groupData.default_cover : groupData.image" /></div>
    </div>
    <section class="connections-wrapper flexWrap container">
        <div class="raw-flex mar-main-sec-cover space-between flexWrap align-center">
            <div class="">
                <div class="flex-start flexWrap">
                    <div v-if="groupData" class="mainnbubbleImgBanner">
                        <img :src="(groupData.icon == null) ? groupData.default_icon : groupData.icon" />
                    </div>
                    <div class="userNamecover">
                        <h3 v-if="groupData" class="">{{groupData.name}}</h3>
                    </div>

                    <div v-if="groupData.member" class="mem-pics-container">
                        <img v-for="member in groupData.member.slice(0, 3)" :src="member.profile_pic">
                        <img v-if="groupData.member && groupData.member.length < 3 && groupOwnerData" :src="getGroupOwnerProfilePic()" />
                        <span v-if="groupData.member" class="enroll-count">{{ groupData.member.length+1 }} Enrolled</span>
                    </div>

                </div>
            </div>

<!--            <button v-if="currentUserGroupData" class="btn-de-default secondary-btn join-g-btn border-btn" type="button" @click="leaveGroup">Leave</button>-->
            <button v-if="!currentUserGroupData && groupData.private != 1" class="btn-de-default secondary-btn join-g-btn" type="button"  @click="joinGroup">Join Group</button>

        </div>
        <div  v-if="(groupData.tags) && (groupData.tags.length > 0)" class="raw-flex flexWrap tagcover">
            <div v-for="tag in groupData.tags.slice(0,4)" class="tagItem">
                <span class="tagText"> {{tag.name}} </span>
            </div>
        </div>
        <div class="tabs-header-wrapper">
            <nav>
                <div class="tabs-header flex-box align-center space-between nav-tabs nav" role="tablist">
                    <a class="tab-link nav-link" data-toggle="tab"  id="about-nav" href="#AboutGroup" role="tab" aria-controls="about" aria-selected="true" @click="updateHash('about')">
                        <span>About</span>
                        <span class="bolder">About</span>
                    </a>
                    <a class="tab-link nav-link active" data-toggle="tab" id="discussion-nav" href="#Discussions" role="tab" aria-controls="discussions" aria-selected="true" @click="updateHash('discussion')">
                        <span>Discussion</span>
                        <span class="bolder">Discussion</span>
                    </a>
                    <a class="tab-link nav-link" data-toggle="tab"  id="event-nav" href="#GroupEvents" role="tab" aria-controls="GroupEvents" aria-selected="true" @click="updateHash('event')">
                        <span>Meetings</span>
                        <span class="bolder">Meetings</span>
                    </a>
                    <a class="tab-link nav-link"  v-if="groupData.is_messenger && groupData.is_messenger == 1" id="messenger-nav" href="/connect/user/messenger">
                        <span>Messenger</span>
                        <span class="bolder">Messenger</span>
                    </a>
<!--                    <a class="tab-link nav-link"  v-if="groupData.is_messenger && groupData.is_messenger == 1" data-toggle="tab"  id="messenger-nav" href="#GroupMessenger" role="tab" aria-controls="GroupMessenger" aria-selected="true" @click="updateHash('messenger')">
                        <span>Messenger</span>
                        <span class="bolder">Messenger</span>
                    </a>-->
                </div>
            </nav>
        </div>
    </section>
</div>
</template>

<script>
import {mapActions} from "vuex";
import Axios from "../../store/axios";

export default {
    name: 'groupHeader',
    props: ["groupData", "groupOwnerData", "currentUserGroupData"],
    data() {
        return {
            s3Url: "https://thinknative.s3-us-west-2.amazonaws.com/"
        };
    },
    mounted : function () {
        //Catch hash from url and open section for the user
        if(this.$router.currentRoute.hash !== '') {
            var hashData = this.$router.currentRoute.hash.slice(1) + '-nav';
            var messengerNav =  document.getElementById('messenger-nav');
            var groupMessengerNav =  document.getElementById('GroupMessenger');

            document.getElementById('about-nav').classList.remove("active")
            document.getElementById('AboutGroup').classList.remove("show", "active")
            document.getElementById('discussion-nav').classList.remove("active")
            document.getElementById('Discussions').classList.remove("show", "active")
            document.getElementById('event-nav').classList.remove("active")
            document.getElementById('GroupEvents').classList.remove("show", "active")

            //Check for messenger tab
            if (typeof(messengerNav) != 'undefined' && messengerNav != null)
            document.getElementById('messenger-nav').classList.remove("active")

            if (typeof(groupMessengerNav) != 'undefined' && groupMessengerNav != null)
            document.getElementById('GroupMessenger').classList.remove("show", "active")


                if (hashData == 'about-nav') {
                    document.getElementById('about-nav').classList.add("active")
                    document.getElementById('AboutGroup').classList.add("show", "active")
                } else if(hashData == 'discussion-nav') {
                    document.getElementById('discussion-nav').classList.add("active")
                    document.getElementById('Discussions').classList.add("show", "active")
                } else if(hashData == 'event-nav') {
                    document.getElementById('event-nav').classList.add("active")
                    document.getElementById('GroupEvents').classList.add("show", "active")
                } else if(hashData == 'messenger-nav') {
                    if (typeof(messengerNav) != 'undefined' && messengerNav != null)
                    document.getElementById('messenger-nav').classList.add("active")
                    if (typeof(groupMessengerNav) != 'undefined' && groupMessengerNav != null)
                    document.getElementById('GroupMessenger').classList.add("show", "active")
                } else {
                    document.getElementById('about-nav').classList.add("active")
                    document.getElementById('AboutGroup').classList.add("show", "active")
                }

            // document.getElementById(hashData).click();
        }
    },
    methods: {
        getGroupOwnerProfilePic: function () {
            return (this.groupOwnerData.profile_pic);
        },
        groupCoverPoistion: function () {
            return (this.groupData.cover_position && this.groupData.cover_position > 0) ? {top:this.groupData.cover_position} : {top:0};
        },
        leaveGroup: function () {
            this.$confirm({
                message: `Are you sure you want to leave this group?`,
                button: {
                    no: "Cancel",
                    yes: "Yes"
                },
                /**
                 * Callback Function
                 * @param {Boolean} confirm
                 */
                callback: confirm => {
                    if (confirm) {
                        this.$emit('leaveGroup', true);
                    }
                }
            });
        },
        joinGroup: function () {
            this.$emit('leaveGroup', false);
        },
        updateHash: function (hashValue) {
            this.$router.push({ hash: hashValue });
            if(hashValue == 'messenger') this.$emit('updateMessengerData', true);
            else this.$emit('updateMessengerData', false);
        }
    }
}
</script>

<style scoped lang="scss">
@import "resources/assets/sass/abstracts/_variables.scss";

.tabs-header-wrapper {
    margin-top: 30px;
    border-top: $light-card-border;
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
            padding: 10px 15px 8px;
            white-space: nowrap;
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
        padding: 10px 15px 8px;
        white-space: nowrap;

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
            bottom: 7px;
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
