<template>
    <main class = "emails-page groups-messenger-wrapper">
        <section class = "page-title-container relative">
            <template>
            <h2 class = "page-title">Messenger</h2>
            <span class = "new-message">
                <span>New Message</span>
                <i class = "icon-add icon-round-bg-small cursorPointer" data-toggle = "modal" data-backdrop = "static"
                   data-keyboard = "false" data-target = "#createMsgModal"></i>
            </span>
            </template>
            <a v-if="tabletMode && openEmailFlag" 
               href="#"
               @click.prevent="backToEmail()"
               class="back-nav inPage align-center"
            >
                <i class="icon-arrow-right-round"></i>
                <span>Messages</span>
            </a>
        </section>
        <section class = "emails-container">
            <div v-if = "conversations && conversations.length > 0" class = "container">
                <div v-if="tabletMode && !openEmailFlag || !tabletMode"
                     v-bind:v-scrollbar="!tabletMode"
                     class = "emails-list-col matchSc"
                >
                    <div>
                        <div class = "mail-item cursorPointer"
                             v-for = "(conversation, index) in conversations"
                             :key = "index"
                             @click = "openConversation(index, conversation.id); openEmailTab()"
                             :class = "['conversation-'+conversation.id, { unread: conversation.is_seen == 0 }, { active: conversation.id === activeConversation }]">
                            <div class = "mail-title">
                                <span class = "mail-subject">{{ conversation.name }}</span>
                            </div>
                            <p class = "mail-body-summary" id = "myString">
                                {{ getMessageBody(conversation.last_message) }}
                            </p>
                            <span class = "time-stamp">
                                <time-ago :refresh = "1" long :datetime = "conversation.created_at"></time-ago>
                            </span>
                        </div>
                    </div>
                </div>
                <div class = "flex-1-66 min-width-0" v-if="!tabletMode || tabletMode && openEmailFlag">
                    <div class = "email-detail-col">
                        <div v-if = "threadHeaderMembers && threadHeaderMembers.length > 0"
                             class = "participants-wrapper">
                            <h5>Current Thread Members:</h5>
                            <div v-scrollbar = {alwaysShowTracks:true}>
                                <div class = "horizontal-participant-container">
                                    <div class = "participant-wrapper" v-for = "(member, index) in threadHeaderMembers"
                                         :key = "index">
                                        <img
                                            :src = "member.headshot">
                                        <span>{{ member.full_name }}</span>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class = "max-scroll" v-on:scroll = "loadMore">
                            <div class = "pad-b-12">
                                <div v-if = "threadMessages && threadMessages.length > 0"
                                     v-for = "(message, index) in threadMessages"
                                     :key = "index" class = "email-wrapper"
                                     :class = "[message.user_id == authUserId ? 'my-message-wrapper' : '', 'message-'+message.id]">
                                    <div class = "email-header">
                                        <div class = "from-user-wrapper">
                                            <img :src = "message.user && message.user.headshot ? message.user.headshot : ''"
                                                 alt = ""
                                                 class = "profile-img small" />
                                            <span class = "from-username">
                                                {{ message.user && message.user.full_name ? message.user.full_name : '' }}
                                            </span>
                                        </div>
                                        <span class = "time-stamp">
                                            <time-ago :key = "threadMessages.id" :refresh = "1" long
                                                      :datetime = "message.message_received_time"></time-ago>
                                        </span>
                                    </div>
                                    <div class = "email-content-wrapper">
                                        <p class = "email-body" v-html = "message.body"></p>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div class = "email-reply-box" v-if = "activeConversation">
                            <form @submit.prevent = "submit" @reset.prevent = "reset" ref = "form">
                                <FormElements elem = "ck"
                                              class = "ck-replybox"
                                              placeholder = "Type your message"
                                              :value = "formData.message" name = "message"
                                              v-on:val = "bindValue" />
                                <button type = "submit" :disabled = "formData.message === ''" class = "btn btn-save-note">
                                    Send
                                </button>
                            </form>
                        </div>
                    </div>
                </div>
            </div>
            <div v-else class = "goal-container mar-t-20">
                <p class = "pl-0">You don't have any conversation just yet!</p>
                <hr>
            </div>
        </section>
        <new-conversation :groupData = "groupData" :selectedUserProfile = "selectedUserProfile"
                          @fetchAllConversations = "fetchAllConversations" />
    </main>
</template>

<script>
    import { mapGetters } from "vuex";
    import FormElements from '../../components/partials/FormElements';
    import service from '../../services/groups';
    import to from "await-to-js";
    import NewConversation from "./NewConversation";


    export default {
        name : "GroupMessenger",
        props : [ "groupData", "selectedUserProfile", "updateMessengerListing", "renderMessenger", "shouldShowExistingConvClick", "messageParam", "threadParam" ],
        components : {
            NewConversation,
            FormElements,
        },
        data () {
            return {
                app_url : app_url,
                activeConversation : null,
                formData : {
                    message : ''
                },
                messageOffset : 10,
                messageCount : 0,
                isLoader : true,
                windowWidth: window.outerWidth,
                tabletMode: false,
                openEmailFlag: false
            }
        },
        computed : {
            ... mapGetters ( [ "authUserId", "user", "threadMessages", "threadHeaderMembers", "conversations" ] ),
        },
        watch : {
            //This will be called when prop updated on page load
            groupData : function ( newVal, oldVal ) {
                if ( this.renderMessenger ) this.getGroupConversationsList ();
            },
            windowWidth(width) {
                if (width <= 768) this.tabletMode = true
                else this.tabletMode = false
            },
            updateMessengerListing : function ( newVal, oldVal ) {
                // if(newVal === true) this.getGroupConversationsList ();
            },
            renderMessenger : function ( newVal, oldVal ) {
                if ( newVal === true ) this.getGroupConversationsList ();
            },
            shouldShowExistingConvClick : function ( newVal, oldVal ) {
                if ( typeof ( newVal ) != 'undefined' && newVal > 0 ) {
                    this.isLoader = false;
                    $ ( '.conversation-' + newVal ).trigger ( 'click' );
                    this.isLoader = true;
                }
            }
        },
        created : function () {
            if ( this.groupData && this.groupData.id && this.renderMessenger ) {
                this.getGroupConversationsList ();
            }
        },
        mounted () {
            this.$nextTick(() => {
                window.addEventListener('resize', this.setWidth);
                this.windowWidth <= 768 ? this.tabletMode = true : this.tabletMode = false;
                // console.log(this.windowWidth)
            })
        },
        beforeDestroy() { 
            window.removeEventListener('resize', this.setWidth); 
        },
        methods : {
            setWidth (){
                this.windowWidth = window.outerWidth
            },
            bindValue ( obj ) {
                this.formData[ obj.ref ] = obj.value;
            },
            openEmailTab () {
                this.openEmailFlag = true;
                console.log(this.openEmailFlag)
            },
            backToEmail () {
                this.openEmailFlag = false;
                console.log(this.openEmailFlag);
                console.log(this.tabletMode);
            },
            async getGroupConversationsList () {
                this.showLoader ();
                const [ err, res ] = await to ( service.getRequest ( 'messenger/coversations/list/' + this.groupData.id ) );
                if ( ! err ) {
                    // this.conversations = res.data.conversations;
                    await this.$store.dispatch ( "conversations", res.data.conversations );
                    await this.$store.commit ( "threadMessages", [] );
                    await this.$store.commit ( "threadMessages", res.data.messages_list.data.messages );
                    await this.$store.commit ( "threadHeaderMembers", res.data.messages_list.data.members );
                    this.activeConversation = res.data.conversations[ 0 ].id;
                    this.messageCount = res.data.messages_list.data.messageCount;
                    this.$emit ( 'conversationList', this.conversations );
                    this.$nextTick ( function () {
                        if ( this.threadParam == null || this.messageParam == null ) return;
                        if ( $ ( this.threadParam ).length === 0 ) this.scrollIntoView ();
                        else $ ( this.threadParam )[ 0 ].click ();
                    } );
                    setTimeout ( x => {
                        if ( this.conversations[ 0 ].is_seen == 0 ) {
                            this.$store.commit ( "readConversation", 0 );
                        }
                    }, 2000 );
                    if(!this.tabletMode ) {
                        this.scrollToBottom ();
                    }
                } else {
                    await this.$store.dispatch ( "conversations", [] );
                    await this.$store.commit ( "threadMessages", [] );
                    await this.$store.commit ( "threadHeaderMembers", [] );
                }
                this.hideLoader ();
            },
            getMessageBody ( body ) {
                if ( body == null || body == '' ) return body;
                body = this.stripTags ( body );
                return body.length > 50 ? body.substring ( 0, 50 ) + '...' : body;
            },
            stripTags ( text ) {
                return text.replace ( /(<([^>]+)>)/ig, '' ).replace ( /&nbsp;/ig, " " );
            },
            scrollToBottom () {
                $ ( ".email-detail-col .max-scroll" ).scrollTop ( $ ( ".email-detail-col .max-scroll" )[ 0 ].scrollHeight );
            },
            async openConversation ( index, conversationId ) {
                
                if ( this.isLoader == true ) this.showLoader ();
                const [ err, res ] = await to ( service.getRequest ( 'messenger/coversations/' + conversationId + '/messages' ) );
                if ( ! err ) {
                    await this.$store.commit ( "threadMessages", res.data.messages );
                    await this.$store.commit ( "threadHeaderMembers", res.data.members );
                    this.activeConversation = conversationId;
                    this.messageCount = res.data.messageCount;
                    this.messageOffset = res.data.messages.length;
                    await this.$store.commit ( "readConversation", index );
                    this.scrollIntoView ();
                    if ( this.messageParam != null ) {
                        setTimeout ( x => {
                            let top = $ ( this.messageParam ).offset ();
                            if ( typeof ( top ) == 'undefined' ) return;
                            $ ( ".email-detail-col .max-scroll" ).animate ( { scrollTop : top.top }, 750 );
                            $ ( this.messageParam ).addClass ( "highlighted2" );
                        }, 1500 );
                    }
                    if(!this.tabletMode ) {
                        this.scrollToBottom ();
                    }
                } else {
                    console.log ( err );
                }
                if ( this.isLoader == true ) this.hideLoader ();
                
            },
            async fetchAllConversations ( flag, response, type = 'conversation' ) {
                let index = this.conversations.findIndex ( ( item ) => item.id === response.id ),
                    message = response.message;
                await this.$store.dispatch ( 'updateConversation', { key : index, data : response } );
                if ( typeof ( message.existing_thread ) != 'undefined' && message.existing_thread == true ) {
                    this.activeConversation = response.id;
                }
                if ( type == 'conversation' ) {
                    this.openConversation ( 0, response.id );
                }
            },
            async submit () {
                this.showLoader ();
                const form = this.prepareData ();
                const [ err, res ] = await to ( service.postRequest ( 'messenger', form ) );
                if ( ! err ) {
                    const response = res.data;
                    await this.$store.commit ( "addThreadMessage", response.message );
                    await this.fetchAllConversations ( true, response, 'message' );
                    this.messageOffset = this.messageOffset + 1;
                    this.messageCount = res.data.messageCount;
                    this.success ( 'New Message Created Successfully' );
                    this.resetFormData ();
                    if(!this.tabletMode ) {
                        this.scrollToBottom ();
                    }
                } else {
                    console.log ( err );
                }
                this.hideLoader ();
            },
            prepareData () {
                const form = new FormData ();
                form.set ( "group_id", this.groupData.id );
                form.set ( "thread_id", this.activeConversation );
                form.set ( 'body', this.formData.message );
                return form;
            },
            resetFormData () {
                this.formData.message = "";
            },
            /* loadScrollScript () {
                $ ( '.scroll-content' ).on ( "scroll", function ( e ) {
                    let scrollHeight = $ ( e.target ).height ();
                    //scroll position
                    let scrollPos = $ ( window ).height () + $ ( window ).scrollTop ();
                    // fire if the scroll position is 300 pixels above the bottom of the page
                    if ( ( ( scrollHeight - 300 ) >= scrollPos ) / scrollHeight == 0 ) {
                        console.log ( 123 );
                    }
                } );
            }, */
            async loadMore ( e ) {
                if ( $ ( ".email-detail-col .max-scroll" ).scrollTop () != 0 ) return;
                if ( this.threadMessages.length == this.messageCount || this.threadMessages.length > this.messageCount ) return;
                this.showLoader ();
                let url = 'messenger/coversations/' + this.activeConversation + '/messages?off_set=' + this.messageOffset;
                const [ err, res ] = await to ( service.getRequest ( url ) );
                if ( ! err ) {
                    await this.$store.commit ( "oldMessages", res.data.messages );
                    this.messageOffset = this.messageOffset + res.data.messages.length;
                } else {
                    console.log ( err );
                }
                this.hideLoader ();
            },
            scrollIntoView () {
                $ ( [ document.documentElement, document.body ] ).animate ( {
                    scrollTop : $ ( ".email-detail-col" ).offset ().top
                }, 750 );
            }
        },
        beforeRouteLeave ( to, from, next ) {
        }
    }
</script>

<style src = "vue-multiselect/dist/vue-multiselect.min.css"></style>

<style scoped lang = "scss">
    @import "resources/assets/sass/abstracts/_variables.scss";

    .participants-wrapper {
        margin-bottom: 12px;

        h5 {
            font-family: $semiBold;
            font-size: 1em;
            margin-bottom: 0;
            padding: 15px 15px 0;
        }
    }

    .horizontal-participant-container {
        display: flex;
        max-width: 100%;
        padding: 15px;

        .participant-wrapper {
            display: flex;
            align-items: center;
            padding-right: 10px;
            flex-shrink: 0;
            position: relative;

            &:not(:first-of-type) {
                padding-left: 10px;
            }

            &:not(:last-of-type):after {
                content: "";
                position: absolute;
                right: 0;
                top: 50%;
                transform: translateY(-50%);
                height: 20px;
                width: 1px;
                background: #d8d8d8;
            }

            img {
                width: 40px;
                height: 40px;
                object-fit: cover;
                border-radius: 50%;
                border: 1px solid #d4d4d4;
                margin-right: 15px;
            }
        }
    }

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
                margin-left: -3px
            }
        }

        .enroll-count {
            font-size: 12px;
            color: color(lighter);
            font-family: $semiBold;
            margin-left: 8px;
        }
    }

    .events-wrapper {
        max-width: 670px;
        margin: 0 auto;

        .goal-container {
            background: color(white);
            border-radius: $border-radius;
            box-shadow: $card-shadow;
            padding: 40px;
            margin-bottom: 23px;

            .note-wrapper,
            .ml-main-task {
                margin-left: 54px;
                margin-top: 24px;
            }

            .colorNot {
                font-size: 16px;

                &.mar-t-b-12 {
                    margin: 12px 0;
                }
            }

            .due-d-container {
                .colorNot {
                    margin: 0 0 4px;
                }

                i[class*="icon-"] {
                    color: lighten(color(dark), 5%);
                }

                .column-wise {
                    flex-direction: column;
                    justify-content: flex-end;
                }
            }

            .ProfileNameTaskss {
                font-size: 16px;
                color: color(primary);
                font-family: $bold;
            }

            .p-note {
                padding: 10px 0;
                font-size: 18px;
                color: color(dark);

                &.p-task {
                    margin: 0;
                }

                p {
                    margin-bottom: 0;
                    color: color(dark);
                }

                &.p-event {
                    border-bottom: $borderD8;
                }
            }

            .iconCalenderTask {
                i {
                    font-size: 28px;
                    color: color(secondary);
                }
            }

            &:last-of-type {
                margin-bottom: 33px;
            }

            .dropdown-toggle {
                font-size: 6px;
            }

            .goal-title-container {
                margin-left: 24px;

                .goal-title {
                    font-size: 22px;
                    color: color(newLight);

                    &.note-h {
                        font-family: $bold;
                    }
                }

                .createdGoalsdate {
                    color: color(brndPrim);
                    font-size: 16px;
                    margin-top: 4px;
                }

                p {
                    margin-bottom: 0;
                }
            }

            .bgGoalPadding {
                margin-left: 54px;

                p {
                    margin-bottom: 0;
                    font-size: 18px;
                    color: color(dark);
                }
            }
        }
    }

    .expand-btn-holder {
        display: block;
        padding: 0;
    }

    .rsvp-wrapper {
        display: flex;
        flex-wrap: wrap;
        padding: 10px 0;
        margin-right: -10px;
        margin-left: -10px;
        max-height: 300px;
        overflow-y: auto;

        h6 {
            font-family: $bold;
            font-size: 18px;
            padding: 0 10px;
        }

        .rsvp-col {
            display: flex;
            align-items: center;
            margin-top: 10px;

            span {
                margin-left: 10px;
                font-family: $semiBold;
                color: color(light);
                white-space: nowrap;
                overflow: hidden;
                text-overflow: ellipsis;
                max-width: 110px;
            }
        }
    }

    .round-img {
        width: 40px;
        height: 40px;
        position: relative;
        overflow: hidden;
        border-radius: 50%;

        img {
            min-width: 100%;
            min-height: 100%;
            @include centralize-absolute;
            object-fit: cover;
        }
    }

    .secondary-color {
        i.icon-dots {
            color: color(secondary);
        }
    }

    div#myMeetingDiv {
        margin-left: 0;
    }

    @include mobile {
        .emails-page .emails-container .container {
            padding: 0;
        }
    }

    @media (min-width: 769px) {
        .page-title-container {
            justify-content: space-between;
            align-items: center;

            .page-title {
                flex: 0 1 auto;
            }
        }
    }
</style>
