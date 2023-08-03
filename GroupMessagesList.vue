<template>
    <div class="email-detail-col">
                    <div class="email-wrapper">
                        <div class="email-header">
                            <div class="from-user-wrapper">
                                <img src="" alt=""
                                     class="profile-img small"/>
                                <span class="from-username">
                                    From User Full Name
                                </span>
                            </div>
                            <span class="time-stamp">
                              Time stamp
                          </span>
                            <!-- <div class="email-actions cursorPointer">
                                <i class="icon-respond"></i>
                            </div> -->
                        </div>
                        <div class="email-content-wrapper">
                          <span class="email-subject">
                              Message Subject
                          </span>
                            <p class="email-body">
                                MEssage Body
                            </p>
                        </div>
                    </div>
                    <div class="email-reply-box">
                        <input type="text" class="width-100" placeholder="Reply"/>
                        <!-- <div class="share-links flex-box just-center mt-3">
                            <div class="cursorPointer">
                                <a class="google_plus customer share blocked text-center" @click="redirectToGoogle">
                                    <i class="fa fa-google-plus blocked">&nbsp;</i>
                                    <span class="blocked ">Gmail</span>
                                </a>
                            </div>
                        </div> -->
                    </div>
                </div>
</template>

<script>
import {mapGetters} from "vuex";
import FormElements from '../../components/partials/FormElements';
import service from '../../services/groups';
import to from "await-to-js";

export default {
    name: "GroupMessenger",
    props: ["groupData"],
    components : { FormElements },
    data () {
        return {
            app_url : app_url,
            value: [
                { name: 'Javascript', code: 'js' }
            ],
            options: [
                { name: 'Vue.js', code: 'vu' },
                { name: 'Javascript', code: 'js' },
                { name: 'Open Source', code: 'os' }
            ],
            conversations : [],
        }
    },
    computed: {
        ...mapGetters(["authUserId", "user"]),
    },
    watch: {
        //This will be called when prop updated on page load
        groupData: function(newVal, oldVal) {
            this.getGroupConversationsList();
        }
    },
    updated: function () {

    },
    created: function () {
        if(this.groupData && this.groupData.id) {
            this.getGroupConversationsList();
        }
    },
    methods: {
        async getGroupConversationsList (){
            this.showLoader();
            const [ err , res ] = await to ( service.getRequest('messenger/coversations/list/'+this.groupData.id ) );
            if ( !err ){
                this.conversations = res.data;
                //await this.$store.commit("groupEvents", res.data);
            }else{
                console.log(err);
            }
            this.hideLoader();
        },
    },
    beforeRouteLeave(to, from, next) {
    }
}
</script>
