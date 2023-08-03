<template>
    <div class = "modal fade" id = "createMsgModal" role = "dialog" aria-hidden = "true">
        <div class = "modal-dialog modal-lg taskEditor" role = "document">
            <div class = "modal-content">
                <div class = " modalEditor taskEditorModal">
                    <div class = "d-flex justify-content-between align-center">
                        <h5 class = "modal-title modalEditorTitle">
                            <span>Send {{groupsLabel}} Message</span>
                        </h5>
                        <button type = "button" class = "close CloseBtn" data-dismiss = "modal" aria-label = "Close"
                                @click = "resetFormData">
                            <i class = "icon-close"></i>
                        </button>
                    </div>
                </div>

                <ValidationObserver v-slot = "{ passes, reset }">
                    <form @submit.prevent = "passes(submit)" @reset.prevent = "reset" ref = "form">
                        <div class = "modal-body">
                            <div>
                                <ValidationProvider rules = "required" class = "vee-validate"
                                                    name = "conversation_users"
                                                    v-slot = "{ errors, required, ariaInput, ariaMsg }">

                                    <multiselect v-model = "formData.conversation_users" :options = "members"
                                                 :multiple = "true" :close-on-select = "false"
                                                 :clear-on-select = "false" :preserve-search = "true"
                                                 :tag-placeholder = "getPlaceHolderText()"
                                                 :placeholder = "getPlaceHolderText()" label = "name"
                                                 track-by = "code" :preselect-first = "true"
                                                 :class = "{ 'has-error-input': errors[0] }" :option-height = "104">
                                        <template slot = "singleLabel" slot-scope = "props">
                                            <img class = "option__image" :src = "props.option.img" />
                                            <span class = "option__desc">
                                                <span class = "option__title">{{ props.option.name }}</span>
                                            </span>
                                        </template>
                                        <template slot = "option" slot-scope = "props">
                                            <img class = "option__image" :src = "props.option.img" />
                                            <div class = "option__desc">
                                                <span class = "option__title">{{ props.option.name }}</span>
                                            </div>
                                        </template>
                                    </multiselect>
                                    <span class = "has-error-span" v-bind = "ariaMsg" v-if = "errors[0]">
                                        Please select members
                                    </span>
                                </ValidationProvider>
                            </div>
                        </div>
                        <div class = "modal-body noteEditorBodyPadding ">
                            <div class = "row mt-3">
                                <div class = "col-md-12">
                                    <div class = "maxHeightEditor">
                                        <FormElements rules = "required" elem = "ck"
                                                      placeholder = "Type your message."
                                                      :value = "formData.message" name = "message"
                                                      v-on:val = "bindValue" />
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div class = "modal-footer-padding">
                            <div class = "d-flex justify-content-start">
                                <button type = "submit"
                                        :disabled = "formData.message === '' || formData.conversation_users.length == 0"
                                        class = "btn btn-save-note">Send
                                </button>
                                <button type = "reset" class = "hide resetForm">Reset</button>
                            </div>
                        </div>
                    </form>
                </ValidationObserver>
            </div>
        </div>
    </div>
</template>

<script>

    import service from '../../services/groups';
    import to from "await-to-js";
    import Multiselect from 'vue-multiselect'
    import FormElements from "../../components/partials/FormElements";
    import {mapGetters} from "vuex";

    export default {
        name : 'NewConversation',
        props : [ "groupData", "selectedUserProfile" ],
        components : {
            FormElements,
            Multiselect
        },
        data () {
            return {
                members : [],
                formData : {
                    conversation_users : [],
                    message : ''
                }
            };
        },
        computed: {
            ...mapGetters(["groupsLabel"]),
        },
        watch : {
            groupData : function ( newVal, oldVal ) {
                this.fetchAllMembers ();
            },
            selectedUserProfile : function ( newVal, oldVal ) {
                this.setConversationParticipant ();
            }
        },
        created () {
            if ( this.groupData && this.groupData.id ) {
                this.fetchAllMembers ();
            }
        },
        methods : {
            bindValue ( obj ) {
                this.formData[ obj.ref ] = obj.value;
            },
            customLabel ( { name, code } ) {
                return `${ name }`
            },
            resetFormData () {
                this.formData.message = "";
                this.formData.conversation_users = [];
                $ ( '.resetForm' ).click ();
            },
            submit () {
                this.showLoader ();
                const form = this.prepareData ();
                let url = "/connect/messenger";
                axios.post ( url, form )
                    .then ( ( res ) => {
                        const response = res.data.data;
                        this.$emit ( 'fetchAllConversations', true, response );

                        if(res.data.is_new_conversation === true) this.success ( 'New Conversation Created Successfully' );
                        this.resetFormData ();
                        $ ( '#createMsgModal' ).modal ( 'toggle' );
                        this.hideLoader ();
                    } ).catch ( err => {
                    this.error ( 'Something went wrong. Please try again later' );
                    this.hideLoader ();
                } );
            },
            prepareData () {
                let users = [];
                this.formData.conversation_users.forEach ( ( item, k ) => {
                    users.push ( item.code )
                } )
                const form = new FormData ();
                form.set ( "group_id", this.groupData.id );
                form.set ( 'users', users.toString () );
                form.set ( 'body', this.formData.message );
                return form;
            },
            async fetchAllMembers () {
                this.showLoader ();
                const [ err, res ] = await to ( service.getRequest ( '/groups/members/' + this.groupData.id ) );
                if ( ! err ) {
                    this.members = res.data.members;
                } else {
                    console.log ( err );
                }
                this.hideLoader ();
            },
            setConversationParticipant : function () {
                this.formData.conversation_users.push (
                    {
                        code : this.selectedUserProfile.id,
                        name : this.selectedUserProfile.full_name
                    }
                );
            },
            getPlaceHolderText : function () {
                return "Select " + this.groupsLabel + " Members";
            }
        }
    }
</script>

<style lang = "scss" scoped>
    @import "resources/assets/sass/abstracts/_variables.scss";

    .option__desc, .option__image {
        display: inline-block;
        vertical-align: middle;
    }

    .option__image {
        width: 40px;
        height: 40px;
        -o-object-fit: cover;
        object-fit: cover;
        border-radius: 50%;
        border: 1px solid #d4d4d4;
        margin-right: 15px;
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

    .mem-pics-container {
        display: flex;
        align-items: center;

        img {
            position: relative;
            border-radius: 50%;
            width: 25px;
            height: 25px;
            min-width: 25px;
            min-height: 25px;
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

</style>
