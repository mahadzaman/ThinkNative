<template>
    <div class="events-wrapper">
        <div class="raw-flex">
            <button v-if="groupData.have_access || groupData.has_admin_access" class="primary-btn mar-l-auto btn-de-default" data-target="#eventModal" data-toggle="modal" type="button" data-backdrop = "static" data-keyboard = "false">
                Create Meeting
            </button>
        </div>
        <div v-if="groupEvents && groupEvents.length > 0">
            <div v-if="showIframe" id="myMeetingDiv">
                <div class="video-wrap">
                    <div class="video">
                        <iframe v-on:load="zoomIframeLoaded" id="meetingIframe" width="100%" height="500"
                                :src="iframeSource"
                                frameborder="0" gesture="media" allowfullscreen class="mar-t-20"></iframe>
                        <div class="poorConnectionLink"></div>
                    </div>
                </div>
            </div>
            <div class="goal-container mar-t-20" v-for="(groupEvent, index) in groupEvents" :key="index"
                 :class = "'post-' + groupEvent.id ">
                <div class="user-info-wrapper mid-col-mob-task">
                    <div class="d-flex justify-content-between ">
                        <div class="d-flex justify-content-start ">
                            <div class="iconCalenderTask">
                                <i class="icon-calender eventIcon"></i>
                            </div>
                            <div class="goal-title-container">
                                <p class="goal-title note-h">
                                    {{ groupEvent.body ? groupEvent.body : 'N/A' }}
                                </p>
                            </div>
                        </div>
                        <div v-if="groupEvent.user_id == authUserId">
                            <div class="imgDots d-lg-none d-md-none d-xl-none secondary-color">
                                <div
                                    class="btn-group dropleft removeArrowDown">
                                    <button
                                        aria-expanded="false"
                                        aria-haspopup="true"
                                        class="btn btn-secondary dropdown-toggle dropdown-toggle-split"
                                        data-toggle="dropdown"
                                        type="button">
                                        <i class="icon-dots"></i>
                                    </button>
                                    <div
                                        aria-labelledby="dropdownMenuLink"
                                        class="dropdown-menu">
                                        <a class="dropdown-item" href="#" v-on:click = "removeGroupEvent(groupEvent.id)">
                                            <i class="fa fa-trash mr-1"></i>
                                            Delete
                                        </a>
                                        <a href="#" v-if="!isEventExpired(groupEvent)" data-toggle="modal" data-target="#eventModal"
                                           class="dropdown-item" @click="editGroupEvent( groupEvent )">
                                            <i aria-hidden="true" class="fa fa-pencil-square-o mr-1"></i>
                                            Edit
                                        </a>
                                    </div>
                                </div>
                            </div>
                            <div class="imgDots d-none d-md-block secondary-color">
                                <div
                                    class="dropdown show removeArrowDown">
                                    <a aria-expanded="false"
                                        aria-haspopup="true"
                                        class="btn btn-secondary dropdown-toggle pr-0"
                                        data-toggle="dropdown"
                                        href="#"
                                        id="dropdownMenuLink"
                                        role="button">
                                        <i class="icon-dots"></i>
                                    </a>

                                    <div
                                        aria-labelledby="dropdownMenuLink"
                                        class="dropdown-menu">
                                        <a class="dropdown-item"
                                            href="#"
                                           v-on:click = "removeGroupEvent(groupEvent.id)"
                                        ><i
                                            class="fa fa-trash mr-1"></i>Delete</a>
                                        <a href="#" data-toggle="modal" v-if="!isEventExpired(groupEvent)" data-target="#eventModal"
                                           class="dropdown-item" @click="editGroupEvent( groupEvent )">
                                            <i aria-hidden="true" class="fa fa-pencil-square-o mr-1"></i>
                                            Edit
                                        </a>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="ml-main-task ">
                        <div class="border-bottom-checkTask">
                            <p class="checkTask ">
                                {{ groupEvent.title ? groupEvent.title : 'N/A' }}
                            </p>
                        </div>
                        <div class="row mar-t-15">
                            <div class="col-md-5">
                                <div class="d-flex align-center">
                                    <div class="taskProfilePic" v-if="groupEvent.createdby">
                                        <img :src="groupEvent.createdby.headshot"/>
                                    </div>
                                    <div class="mar-l-12">
                                        <h6 class="mb-0">
                                            <!--Role id is not required so we have to identify all places where we are using viewMemberProfile function-->
                                            <span class="meetingBold custom-upload" @click="viewMemberProfile(groupEvent.createdby.id, 'not_required')">
                                                {{ groupEvent.createdby && groupEvent.createdby.full_name ? groupEvent.createdby.full_name : '' }}
                                            </span>
                                            <span class="LftNot LftMeetingLftMeeting">
                                                scheduled a meeting
                                            </span>
                                            <div class="mem-pics-container" v-if="groupEvent.users && groupEvent.users.length > 0">
                                                <img v-for="(user, userIndex) in groupEvent.users.slice(0, 3)" :key="userIndex" :src="user.headshot ? user.headshot : '/img/profile-pic.jpeg'">
                                                <span class="enroll-count">{{ groupEvent.users.length }} {{ groupEvent.users.length > 1 ? 'Members' : 'Member'}}</span>
                                            </div>
                                        </h6>
                                    </div>
                                </div>
                            </div>
                            <div class="col-md-3">
                                <div>
                                    <div class="form-group">
                                        <p class="checkTask ">
                                            Start Time
                                        </p>
                                        <div class="select_box StartTineP">
                                            {{ groupEvent.start_date_time_zone }}
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-md-4">
                                <template v-if="!isEventExpired(groupEvent)">
                                    <div class="vert-row">
                                        <div class="vert-col">
                                            <button v-if="groupEvent.is_r_s_v_p && groupEvent.is_r_s_v_p.length || groupEvent.user_id == authUserId || checkStartTime(groupEvent.start_date_time)"
                                                class="btn btnJoinMetting" id="ShowVideo"
                                                @click.prevent="joinMeeting(groupEvent.zoom_meeting_id, groupEvent.user_id, groupEvent.password)">
                                                Join Meeting
                                            </button>
                                            <button v-else class="btn btnJoinMetting" @click.prevent="addAsRSVP(groupEvent.id, groupEvent.zoom_meeting_id)">
                                                RSVP
                                            </button>
                                        </div>
                                        <div class="vert-col">
                                            <a href="javascript:;" class="bordered-btn copy-clip" title="Copy meeting link" @click="copyToClipboard(groupEvent.join_url , 'Meeting link copied')">
                                                <i class="icon-copy-link"></i>
                                            </a>
                                        </div>
                                    </div>

                                </template>
                                <div v-else class="raw-flex width-100 just-end">
                                    <button class="btn btnJoinMetting" id="ShowVideo" disabled>
                                        Expired
                                    </button>
                                </div>
                            </div>
                        </div>
                        <div v-if="groupEvent.users && groupEvent.users.length > 0">
                            <div class="text-right expand-btn-holder mar-t-10 collapsed">
                                <a href="javascript:void(0)" v-on:click="expandRSVP($event)" class="link-secondary raw-flex align-center">View RSVP's<i class="mar-l-5 icon-arrow-down"></i></a>
                            </div>
                            <div class="rsvp-slide-up-down hide">
                                <div class="rsvp-wrapper">
                                    <h6 class="flex-100">RSVP's list</h6>
                                    <div class="rsvp-col flex-col-50" v-for="(rsvpUser, index) in groupEvent.users">
                                        <div class="round-img">
                                            <img :src="rsvpUser.headshot" alt="">
                                        </div>
                                        <!--Role id is not required so we have to identify all places where we are using viewMemberProfile function-->
                                        <span class="full-name custom-upload" @click="viewMemberProfile(rsvpUser.id, 'not_required')">{{ rsvpUser.full_name ? rsvpUser.full_name : '' }}</span>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div v-else class="goal-container mar-t-20">
            <p v-if="isEventEmpty == false" class="pl-0"></p>
            <p v-else class="pl-0">You don't have any scheduled events just yet!</p>
            <hr>
        </div>

        <div class="modal fade" id="eventModal" tabindex="-1" role="dialog" aria-labelledby="eventModalLabel" aria-hidden="true" data-backdrop="static" data-keyboard="false">
            <div class="modal-dialog modal-lg" role="document">
                <div class="modal-content ">
                    <div class="modal-header modalEditor taskEditorModal raw-flex align-center">
                        <h5 id="noteModalLabel" class="modal-title modalEditorTitle raw-flex align-center">
                            <span>{{ updateFlag == 0 ? 'New Meeting' : 'Update Meeting' }}</span>
                        </h5>
                        <button type="button" data-dismiss="modal" aria-label="Close" class="close CloseBtn" v-on:click = "resetFormData">
                            <i class="icon-close"></i>
                        </button>
                    </div>
                    <ValidationObserver v-slot = "{ passes, reset }">
                        <form @submit.prevent = "passes(submit)" @reset.prevent = "reset" ref = "form">
                            <div class="modal-body meetinModal eventModalPadding">
                                <div class="row">
                                    <div class="col-md-6 col-lg-6 scheduleModalpadding">
                                        <div class="input-group mb-3 meetinModalInput">
                                            <FormElements rules="required|max:100" name="meeting topic" v-on:val="bindValue"
                                                          :value="formData.meeting_topic"
                                                          placeholder="Topic..."/>
                                        </div>
                                    </div>
                                    <div class="col-md-6 col-lg-6 scheduleModalpadding">
                                        <div class="input-group mb-3 meetinModalInput">
                                            <FormElements rules="required|max:180" name="meeting name" v-on:val="bindValue"
                                                          :value="formData.meeting_name"
                                                          placeholder="Agenda of a meeting..."/>
                                        </div>
                                    </div>
                                </div>
                                <div class="row">
                                    <div class="col-md-6 col-lg-6 scheduleModalpadding">
                                        <div>
                                            <div>
                                                <div class="form-group TimeZoneVideoMeeting mt-3">
                                                    <h6 class="colorNot mb-0">Time Zone</h6>
                                                    <div class="select_box_editProfile">
                                                        <FormElements rules="required" placeholder="Timezone"
                                                                      name="timezone" :value="formData.timezone"
                                                                      v-on:val="bindValue" elem="timezoneSelect"
                                                                      :options="timezoneOption"/>
                                                    </div>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="col-md-6 col-lg-6 scheduleModalpadding">
                                        <div class="d-flex justify-content-start mt-2 flex-wrap">
                                            <div class="widthMainDateEvents with100per">
                                                <h6 class="colorNot mb-0">Start Time</h6>
                                                <div class="row">
                                                    <div class="col-md-6 col-lg-6 scheduleModalpadding">
                                                        <div class="d-flex justify-content-start">
                                                            <div class="imgClockTask raw-flex align-center">
                                                                <img src="/img/profileTask/event_not.png"/>
                                                            </div>
                                                            <div class="widthClock ml-2 maxwidthTime">
                                                                <FormElements rules="required" name="start date"
                                                                              :value="formData.start_date"
                                                                              v-on:val="bindValue"
                                                                              elem="date"/>
                                                            </div>
                                                        </div>
                                                    </div>
                                                    <div class="col-md-6 col-lg-6 scheduleModalpadding">
                                                        <div class="d-flex justify-content-start">
                                                            <div class="imgClockTask raw-flex align-center">
                                                                <img src="/img/profileTask/clockTask.png"/>
                                                            </div>
                                                            <div class="widthClock ml-2 maxwidthTime">
                                                                <FormElements rules="required" name="start time"
                                                                              :value="formData.start_time"
                                                                              v-on:val="bindValue"
                                                                              elem="time" dateTime="true"/>
                                                            </div>
                                                        </div>
                                                    </div>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                                <p v-if = "meetingDateTimeError != ''" class = " dateTimeError mt-2">{{ meetingDateTimeError }}</p>
                                <div class="row">
                                    <div class="col-md-6 col-lg-6 scheduleModalpadding">
                                        <div class = "form-check ml-2 mt-2">
                                            <input type = "checkbox" class = "form-check-input chkTaskModal cursorPointer"
                                                   id = "exampleCheck1"
                                                   name = "is_enable_recording"
                                                   v-model = "formData.is_enable_recording"
                                                   @click = "formData.is_enable_recording = !formData.is_enable_recording"
                                            >
                                            <label class = "form-check-label chkmeOut cursorPointer"
                                                   @click = "formData.is_enable_recording = !formData.is_enable_recording">Enable recording for this meeting</label>
                                        </div>
                                    </div>
                                </div>
                            </div>

                            <div class="modal-footer-padding modal-footer-padding-email">
                                <div class="d-flex justify-content-start">

                                    <button type="submit" class="btn  btn-save-note" :disabled="formData.meeting_name && formData.start_date && formData.start_time  && formData.timezone  && formData.meeting_topic  === ''">Save
                                        Meeting
                                    </button>
                                    <button type = "reset" class = "hide resetForm" ref="resetForm">Reset</button>
                                </div>
                            </div>
                        </form>
                    </ValidationObserver>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import {mapGetters} from "vuex";
import to from "await-to-js";
import service from "../../services/groups";
import FormElements from '../../components/partials/FormElements';
import axios from "../../store/axios";

export default {
    name: "GroupEvents",
    props: ["groupData","postParam"],
    components : { FormElements },
    data () {
        return {
            showIframe: false,
            isEventEmpty: false,
            iframeSource: '',
            app_url : app_url,
            paramId : this.$route.params.id,
            formData : {
                meeting_name : '',
                start_date : '',
                start_time : '',
                timezone : '',
                meeting_topic : '',
                is_enable_recording : 0
            },
            timezoneOption : [ ],
            updateFlag : 0,
            meetingDateTimeError : ''
        }
    },
    computed: {
        ...mapGetters(["groupEvents", "authUserId", "user"]),
    },
    watch: {
        //This will be called when prop updated on page load
        groupData: function(newVal, oldVal) {
            this.getGroupEvents();
        }
    },
    updated: function () {

        let self = this;
        /**
         * Leave zoom inprogress meeting before page load
         */
        window.onbeforeunload = function(event)
        {
            self.leaveOrEndInprogressMeeting();
        };

        window.openZoomNativeClient = function (meetingId, pwd) {
            window.open('https://zoom.us/j/'+meetingId+'?pwd='+pwd, '_blank');
        };

        window.closeZoomPage = function (  ) {
            self.showIframe = false;
            self.iframeSource = '';
        }
    },
    created: function () {
        this.fetchTimezones();
        //This will be called when data exist in the store on screen creation time
        if(this.groupData && this.groupData.id){
            this.getGroupEvents();
        }
    },
    methods: {
        expandRSVP: function(e) {
            let expandBtn = $(e.target).closest(".expand-btn-holder");
            let rsvpWrap =  expandBtn.siblings(".rsvp-slide-up-down");
            if (rsvpWrap.is(":visible")) {
                rsvpWrap.slideUp(250);
            } else {
                rsvpWrap.slideDown(250);
            }
        },
        toggleScroll : function ( elem ) {
            setTimeout ( x => {
                let top = $ ( elem ).offset ().top - 175;
                $ ( [ document.documentElement, document.body ] ).animate ( { scrollTop : top }, 750 );
                $ ( elem ).addClass ( "highlighted2" );
            }, 1500 );
        },
        async getGroupEvents (){

            this.showLoader();
            const [ err , res ] = await to ( service.getRequest('groups/'+this.groupData.id+'/events' ) );
            if ( err ){
            }
            else {
                await this.$store.commit("groupEvents", res.data);
                if(res.data && res.data.length == 0){
                    this.isEventEmpty = true;
                } else {
                    if(this.postParam != null){
                        this.toggleScroll(this.postParam);
                        this.postParam = null;
                    }
                }
            }
            this.hideLoader();
        },

        async fetchTimezones() {
            const [ err , res ] = await to ( service.getRequest('timezones' ) );
            if ( err ){
                this.error(err.response.message);
            }
            else {
                this.timezoneOption = res.data.timezones;
            }
        },
        bindValue ( obj ) {
            this.formData[ obj.ref ] = obj.value;
        },
        resetFormData ( event = false ) {
            this.formData.meeting_name = (event) ? event.title : "";
            this.formData.start_date = (event) ? this.$moment(event.start_date) .format("MM/DD/YYYY") : "";
            this.formData.start_time = (event) ? event.start_time : "";
            this.formData.timezone = (event) ? event.timezone : "";
            this.formData.meeting_topic = (event) ? event.body : "";
            this.formData.is_enable_recording = (event) ? event.is_enable_recording : 0;
            if ( !(event.id) ) {
                this.$refs.resetForm.click ();
                this.updateFlag = 0
            }
            this.meetingDateTimeError = '';
        },
        async submit () {
            this.showLoader ();
            const form = this.prepareData ();
            let url = 'groups/posts';
            this.meetingDateTimeError = '';
            if ( this.updateFlag > 0 ) url = url+'/'+this.updateFlag;

            const [ err , res ] = await to ( service.postRequest(url, form ) );
            if ( err ){
                if(err.response.data.message == "You can't create live events in the past."){
                    this.meetingDateTimeError = err.response.data.message;
                }else{
                    this.error ( err.response.data.message );
                }
            }
            else {
                if(this.updateFlag > 0){
                    await this.$store.commit ( 'updateGroupEvent', res.data.post );
                }else{
                    await this.$store.commit ( 'addGroupEvent', res.data.post );
                }
                this.success ( res.message );
                this.resetFormData ();
                $ ( '#eventModal' ).modal ( 'toggle' );
            }
            this.hideLoader();
        },
        async removeGroupEvent ( id ) {
            this.$confirm({
                message: `Are you sure you want to delete this meeting?`,
                button: {
                    no: "Cancel",
                    yes: "Yes"
                },
                callback: async confirm => {
                    if (confirm) {
                        this.showLoader ();
                        const form = {
                            _method : "DELETE",
                            type : 'event'
                        };
                        const [ err , res ] = await to ( service.postRequest('groups/posts/'+id, form ) );
                        if ( err ){
                            this.error ( err.response.data.message );
                        }
                        else {
                            this.$store.commit ( 'removeGroupEvent', id );
                            this.success ( 'Meeting deleted successfully!' );
                        }
                        this.hideLoader();
                    }
                }
            });
        },
        prepareData () {
            console.log(this.groupData.id);
            const form = new FormData ();
            form.set ( "postTitle", this.formData.meeting_name );
            form.set ( "start_date", this.formData.start_date + ' ' + this.formData.start_time );
            form.set ( "postBody", this.formData.meeting_topic );
            form.set ( "timezone", this.formData.timezone );
            form.set ( "is_enable_recording", this.formData.is_enable_recording ? 1 : 0)
            form.set ( "group_id", this.groupData.id );

            if ( this.updateFlag !== 0  ) form.set ( "_method", "PUT" )
            return form;
        },

        joinMeeting (meetingId, createdBy, pwd){

            if (!this.showIframe && this.iframeSource == '') {
                this.showLoader();
                axios.post ( this.authUserId + "/video/meetings/expired/" + meetingId+"/events" )
                    .then ( ( res ) => {
                        const response = res.data;
                        if(response.data.is_expired == 1){
                            this.hideLoader ();
                            this.error ( 'This event has been expired. Please reload your page.', 'Alert' );
                        }else{
                            let role = this.authUserId == createdBy ? 1 : 0;
                            this.showIframe = true;
                            this.iframeSource = this.app_url + "/zoom?meeting=" + meetingId + "&pwd=" + pwd + "&role=" + role + "&is_close=1";
                            setTimeout(function(){
                                document.getElementById("myMeetingDiv").scrollIntoView();
                            }, 3000);
                        }
                    } ).catch ( err => {
                        this.hideLoader ();
                        this.error ( err.response.data.message );
                    } );
            } else {
                this.error('You already have a meeting in progress!');
            }
        },

        async addAsRSVP(postId, zoomId){
            this.showLoader();
            const [ err , res ] = await to ( service.postRequest('groups/'+this.groupData.id+'/post/rsvp', {'post_id' : postId, 'zoom_id' : zoomId} ) );
            if ( err ){
            }
            else {
                if(res.data.is_expired == 1){
                    this.error ( 'This meeting has been expired. Please reload your page.', 'Alert' );
                }else{
                    await this.$store.commit ( 'updateGroupEvent', res.data[0] );
                }
            }
            this.hideLoader();
        },

        /**
         * Leave zoom inprogress meeting
         */
        leaveOrEndInprogressMeeting() {
            if (this.iframeSource && this.showIframe && document.getElementById('meetingIframe').contentWindow && typeof document.getElementById('meetingIframe').contentWindow.endZoomMeeting == 'function') {
                let role = $('.leaveMeeting').length ? $('.leaveMeeting').attr('data-role') : 0;
                document.getElementById('meetingIframe').contentWindow.endZoomMeeting(role);
            }
        },

        checkStartTime(startDate){
            let eventStartDate = Date.parse(startDate);
            let currentDate = Date.parse(new Date());
            if(eventStartDate <= currentDate){
                return true;
            }
            return false;
        },
        editGroupEvent ( event ){
            this.updateFlag = event.id;
            this.resetFormData( event );
        },
        viewMemberProfile: function (userId, roleId) {
            this.$emit('viewMemberProfile', userId, roleId);
        },
        isEventExpired(meeting){
            return meeting.is_expired == 1;
        }
    },
    mounted : function (){
        $('input[id="start date"]').attr("placeholder",this.$moment(new Date()).format("MM/DD/YYYY"));
        $('input[id="start time"]').attr("placeholder",this.$moment(new Date()).format("hh:mm a"));
    },
    beforeRouteLeave(to, from, next) {
        this.leaveOrEndInprogressMeeting();
        next();
    }
}
</script>

<style scoped lang="scss">
@import "resources/assets/sass/abstracts/_variables.scss";

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

            @include mobile {
                margin-left: 0;
            }
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
.secondary-color{
    i.icon-dots{
        color: color(secondary);
    }
}

div#myMeetingDiv {
    margin-left: 0;
}
</style>
