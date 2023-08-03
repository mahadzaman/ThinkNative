<template>
    <section id = "notifications-bar" class = "notification-widget">
        <div class = "notify-wrapper" id = "smoothScroll">
            <div class = "notify-header">
                <h5>Notifications</h5>
                <a href = "#" class = "close-notify"><i class = "icon-close"></i></a>
            </div>
            <div v-if="notifications.length > 0" v-for = "notification in notifications" class = "notification-group">
                <a v-bind:href = "notification.actions && notification.actions.length ? 'javascript:void(0)' : appUrl + '/' + notification.link">
                    <div class = "single-notfication">
                        <div class = "left-col">
                            <span class="alarm-circle">
                                <i class="icon-bell"></i>
                            </span>
                        </div>
                        <div class = "right-col">
                            <span class = "notify-info" v-html="notification.name"></span>
                        </div>
                    </div>
                </a>
                <div v-if="notification.actions" class = "raw-flex pad-10">
                    <a class="mr-1" v-for="(acceptReject ,actionIndex) in notification.actions" :class = "getBtnClass(actionIndex)" v-bind:href="appUrl + '/' + acceptReject.link" :key="actionIndex">
                        {{(actionIndex == 0) ? 'Accept' : 'Reject'}}
                    </a>
                </div>
            </div>
            <div v-if="notifications.length == 0"><label>You do not have any new notifications.</label></div>
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
        name : "notifications-bar",
        data () {
            return {
                appUrl : app_url
            };
        },
        computed : {
            ... mapGetters ( [ "notifications", "latest_notifications" ] )
        },
        methods : {
            ... mapActions ( [ "getNotifications" ] ),
            getBtnClass: function (index) {
                return index == 0 ? 'notify-info primary-btn ml-2' : 'notify-info secondary-btn';
            }
        }
    };
</script>

<style scoped lang = "scss">
    @import "resources/assets/sass/abstracts/_variables.scss";

    .notification-widget {
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

                        .alarm-circle {
                            height: 48px;
                            width: 48px;
                            display: flex;
                            align-items: center;
                            justify-content: center;
                            background: color(card-bg);
                            color: color(icon-color);
                            border-radius: 50%;
                            font-size: 22px;
                            margin-right: 10px;
                        }
                    }

                    .right-col {
                        color: color(light);
                        display: flex;
                        align-items: center;

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
    }
</style>
