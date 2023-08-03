<template>
    <section id = "activities-bar" class = "activity-widget">
        <div class = "notify-wrapper" id = "smoothScroll">
            <div class = "notify-header">
                <h5>Activities</h5>
                <a href = "#" class = "close-notify"><i class = "icon-close"></i></a>
            </div>

            <div v-if="activities.length > 0" class = "notification-group" v-for = "(activity, index) in activities" :key = "index">
                <div class = "noti-group-header">
                    <i :class = "{
                        'icon-notes': Object.keys(activity)[0] == 'tasks',
                        'icon-task': Object.keys(activity)[0] == 'notes',
                        'icon-video': Object.keys(activity)[0] == 'videomeetings',
                        'icon-phone': Object.keys(activity)[0] == 'smslog',
                        'icon-calender': Object.keys(activity)[0] == 'meets',
                        'icon-msg': Object.keys(activity)[0] == 'useremails',
                        'icon-resources': Object.keys(activity)[0] == 'course' || Object.keys(activity)[0] == 'chapter' || Object.keys(activity)[0] == 'lesson'
                    }"></i>
                    <span class = "text-capitalize"
                          v-if = "Object.keys(activity)[0] && Object.keys(activity)[0] == 'useremails'"> Emails </span>
                    <span class = "text-capitalize"
                          v-else-if = "Object.keys(activity)[0] && Object.keys(activity)[0] == 'videomeetings'"> Video Meetings </span>
                    <span class = "text-capitalize"
                          v-else-if = "Object.keys(activity)[0] && Object.keys(activity)[0] == 'smslog'"> Messages </span>
                    <span class = "text-capitalize" v-else> {{ Object.keys(activity)[0] }} </span>
                </div>
                <div class = "single-notfication" v-for = "(item, k) in activity[ Object.keys ( activity )[ 0 ] ]"
                     :key = "k">
                    <div class = "left-col">
                        <img v-if = "item.created_by" :src = "item.created_by.profile_pic" alt = ""
                             class = "user-img" />
                        <img v-else-if = "item.user" :src = "item.user.profile_pic" alt = "" class = "user-img" />
                    </div>
                    <div class = "right-col" v-if = "item.type == 'Tasks'">
                        <router-link v-if = "item.created_by"
                                     :to = "'/tasks/' + item.created_by_type + '/'+ item.created_by.id + '?item_id=' + item.activeable.id">
                            <span v-if = "item.created_by" class = "user-name">{{ item.created_by.full_name }}</span>
                            <span class = "notify-info">left a
                                <span>task</span>
                                for you!
                            </span>
                            <span v-if = "item.time" class = "time-stamp blocked">{{ item.time }}</span>
                        </router-link>
                    </div>
                    <div class = "right-col" v-if = "item.type == 'Notes'">
                        <router-link v-if = "item.created_by"
                                     :to = "'/notes/' + item.created_by_type + '/' + item.created_by.id + '?item_id=' + item.activeable.id"
                                     class = "close-notify">
                            <span v-if = "item.created_by" class = "user-name">{{ item.created_by.full_name }}</span>
                            <span class = "notify-info">left a
                                <span v-if = "item.type == 'Notes'">note</span>
                                for you!
                            </span>
                            <span v-if = "item.time" class = "time-stamp blocked">{{ item.time }}</span>
                        </router-link>
                    </div>
                    <div class = "right-col" v-if = "item.type == 'VideoMeetings'">
                        <router-link v-if = "item.created_by"
                                     :to = "'/video-meetings/' + item.created_by_type + '/' + item.created_by.id + '?item_id=' + item.activeable.id">
                            <span v-if = "item.created_by" class = "user-name">{{ item.created_by.full_name }}</span>
                            <span class = "notify-info">scheduled a
                                <span v-if = "item.type == 'VideoMeetings'">video meeting invite</span>
                                for you!
                            </span>
                            <span v-if = "item.time" class = "time-stamp blocked">{{ item.time }}</span>
                        </router-link>
                    </div>
                    <div class = "right-col" v-if = "item.type == 'SmsLog'">
                        <router-link v-if = "item.created_by"
                                     :to = "'/sms/' + item.created_by_type + '/' + item.created_by.id + '?item_id=' + item.activeable.id">
                            <span class = "user-name">You </span>
                            <span class = "notify-info">have received a
                                <span v-if = "item.type == 'SmsLog'">message</span>
                                from
                                <span v-if = "item.created_by">{{ item.created_by.full_name }}</span>
                            </span>
                            <span v-if = "item.time" class = "time-stamp blocked">{{ item.time }}</span>
                        </router-link>
                    </div>
                    <div class = "right-col" v-if = "item.type == 'Meets'">
                        <router-link v-if = "item.created_by"
                                     :to = "'events/' + item.created_by_type + '/' + item.created_by.id + '?item_id=' + item.activeable.id">
                            <span v-if = "item.created_by" class = "user-name">{{ item.created_by.full_name }}</span>
                            <span class = "notify-info">has scheduled a
                                <span v-if = "item.type == 'Meets'">meeting</span>
                                for you!
                            </span>
                            <span v-if = "item.time" class = "time-stamp blocked">{{ item.time }}</span>
                        </router-link>
                    </div>
                    <div class = "right-col" v-if = "item.type == 'UserEmails'"
                    >
                        <router-link v-if = "item.created_by"
                                     :to = "'/emails/' + item.created_by_type + '/' + item.created_by.id + '?item_id=' + item.activeable.id">
                            <span class = "user-name">You</span>
                            <span class = "notify-info">have received an
                                <span v-if = "item.type == 'UserEmails'">email</span>
                                from
                                <span v-if = "item.created_by">{{ item.created_by.full_name }}</span>
                            </span>
                            <span v-if = "item.time" class = "time-stamp blocked">{{ item.time }}</span>
                        </router-link>
                    </div>
                    <div class = "right-col" v-if = "item.type == 'Course'">
                        <span class = "user-name">You </span>
                        <span class = "notify-info">have {{ item.status == 'read' ? 'viewed' : item.status }} the {{ item.activeable.title }} course</span>
                        <span v-if = "item.time" class = "time-stamp blocked">{{ item.time }}</span>
                    </div>
                    <div class = "right-col" v-if = "item.type == 'Chapter'">
                        <span class = "user-name">You</span>
                        <span class = "notify-info">
                            have {{ item.status == 'read' ? 'viewed' : item.status }} the {{item.activeable.title}} chapter
                        </span>
                        <span v-if = "item.time" class = "time-stamp blocked">{{ item.time }}</span>
                    </div>
                    <div class = "right-col" v-if = "item.type == 'Lesson'">
                        <span class = "user-name">You</span>
                        <span class = "notify-info">
                            have {{item.status == 'read' ? 'viewed' : item.status}} the {{item.activeable.title}} lesson
                        </span>
                        <span v-if = "item.time" class = "time-stamp blocked">{{ item.time }}</span>
                    </div>

                    <div class = "right-col" v-if = "item.type == 'Exam'">
                        <span class = "user-name">You</span>
                        <span class = "notify-info">
                            have {{item.status}} the {{item.activeable.title}} lesson
                        </span>
                        <span v-if = "item.time" class = "time-stamp blocked">{{ item.time }}</span>
                    </div>
                </div>
            </div>
            <div v-if="activities.length == 0"><label>You do not have any activity.</label></div>
        </div>
    </section>
</template>

<script>
    import {
        mapGetters,
        mapActions
    } from "vuex";
    import store from "../../store";

    export default {
        name : "activities-bar",
        data () {
            return {
                appUrl : app_url
            };
        },
        computed : {
            ... mapGetters ( [ "activities" ] )
        },
        watch : {
            activities () {
                return this.$store.getters.activities
            }
        },
        methods : {
            ... mapActions ( [ "getNotifications" ] ),
            roleType : function ( value ){
                if ( !value ) return '';
                else if ( value == 3 ) return 'champion'
                else return 'member';
            }
        }
    };
</script>

<style scoped lang = "scss">
    @import "resources/assets/sass/abstracts/_variables.scss";

    .notification-widget, .activity-widget {
        background: $backdrop;
        position: fixed;
        top: 0;
        right: 0;
        width: 100%;
        height: 100%;
        z-index: 999;
        visibility: hidden;
        opacity: 0;
        transition: visibility ease 0.275s, opacity ease 0.275s;

        &.noti-drawer-open {
            visibility: visible;
            opacity: 1;

            .notify-wrapper {
                right: 0;
            }
        }

        .notify-wrapper {
            position: absolute;
            top: 0;
            right: -488px;
            width: 100%;
            max-width: 488px;
            background: color(white);
            padding: 30px 48px;
            height: 100%;
            overflow-y: auto;
            transition: right 0.275s ease;

            .notify-header {
                display: flex;

                h5 {
                    font-family: $bold;
                    font-size: 32px;
                }

                .close-notify {
                    margin-left: auto;
                }

                .icon-close {
                    border-radius: 50%;
                    padding: 14px;
                    background: #f0f0f0;
                    color: color(dark);
                    margin-left: auto;
                    font-size: 10px;
                }
            }

            .notification-group {
                background: color(bg-main);
                border-radius: $border-radius;
                margin-top: 30px;

                .noti-group-header {
                    display: flex;
                    align-items: center;
                    border-bottom: $light-card-border;
                    padding: 15px 20px;
                    color: color(dark);

                    span {
                        font-size: 18px;
                        font-family: $semiBold;
                        margin-left: 10px;
                        display: inline-block;
                    }

                    i {
                        color: color(secondary);
                        font-size: 22px;
                    }
                }

                .single-notfication {
                    display: flex;
                    padding: 20px 20px 15px;
                    border-bottom: $light-card-border;

                    &:last-of-type {
                        border: none;
                    }

                    .left-col {
                        display: flex;
                        flex-direction: column;
                        align-items: baseline;

                        .notify-type {
                            position: relative;
                            z-index: 1;
                            padding: 10px;
                            border-radius: 50%;
                            background: color(secondary);
                            color: color(white);
                            font-size: 16px;
                        }

                        .notify-type + .user-img {
                            position: relative;
                            margin-left: 20px;
                            left: -7px;
                            top: -20px;
                            z-index: 2;
                            margin-right: 5px;
                            min-width: 36px;
                            min-height: 36px;
                            max-width: 36px;
                            max-height: 36px;
                        }

                        .user-img {
                            min-width: 48px;
                            min-height: 48px;
                            max-width: 48px;
                            max-height: 48px;
                            border-radius: 50%;
                            object-fit: cover;
                            margin-right: 12px;
                        }
                    }

                    .right-col {
                        color: color(light);

                        .user-name,
                        .module-name {
                            font-family: $bold;
                        }

                        .time-stamp {
                            color: color(secondary);
                            font-size: 12px;
                        }
                    }
                }
            }
        }

        .right-col a {
            color: color(dark);

            &:hover {
                color: darken(color(dark), 50%)
            }
        }
    }
</style>
