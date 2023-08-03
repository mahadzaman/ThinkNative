<template>
    <div
        class="modal fade"
        id="createMsgModal"
        role="dialog"
        aria-hidden="true"
    >
        <div class="modal-dialog taskEditor" role="document">
            <div class="modal-content">
                <div class=" modalEditor taskEditorModal">
                    <div class="d-flex justify-content-between align-center">
                        <h5 class="modal-title modalEditorTitle">
                            <span>New Thread</span>
                        </h5>
                        <button
                            type="button"
                            class="close CloseBtn"
                            data-dismiss="modal"
                            aria-label="Close"
                            @click="groupMembersList = false; back($event);"
                        >
                            <i class="icon-close"></i>
                        </button>
                    </div>
                </div>

                <div class="modal-body">
                    <div v-if="groupMembersList" class="mar-b-10 info-nav-bar">
                        <a
                            href="#"
                            class="back-nav blocked mar-b-10"
                            @click.prevent="groupMembersList = false"
                            @click="back($event)"
                        >
                            <i class="icon-arrow-right-round"></i>
                            <span>Back</span>
                        </a>
                        <div class="group-info">
                            <img :src="groupFullInfo.headshot" alt="" />
                            <h6>{{ groupFullInfo.name }}</h6>
                        </div>
                    </div>
                    <div class="raw-flex pad-b-10">
                        <div class="search">
                            <input
                                type="text"
                                placeholder="Search"
                                class="searchTerm"
                                v-model="searchQuery"
                            />
                            <button
                                type="submit"
                                class="searchButton btn-de-default"
                            >
                                <i class="icon-search colorSearch"></i>
                            </button>
                        </div>
                    </div>
                    <div class="users-wrapper row-x-y-5-minus">
                        <template v-if="!groupMembersList">
                            <div
                                v-for="(individual,
                                index) in individualsUsers"
                                :key="index"
                                class="flex-100 pad-5 all-options"
                            >
                                <a href="#" class="user-wrapper" @click="createNewThread(individual.user_id)">
                                    <img
                                        :src="individual.headshot"
                                        alt=""
                                        class="profile-img"
                                    />
                                    <span class="participant-name">
                                        {{ individual.name }}
                                    </span>
                                    <span class="participant-role">{{ individual.role }}</span>
                                </a>
                            </div>
                            <div
                                v-for="(group, index) in allgroups"
                                :key="`group-${index}`"
                                class="flex-100 pad-5 all-options"
                            >
                                <a
                                    href="#"
                                    class="user-wrapper"
                                    @click.prevent="groupSelection(index, group.id)"
                                >
                                    <img
                                        :src="group.headshot"
                                        alt=""
                                        class="profile-img"
                                    />
                                    <span class="participant-name">
                                        {{ groupMessageName(group.name) }}
                                    </span>
                                    <span class="group-info">
                                        {{ group.membersSummary }}
                                    </span>
                                    <span class="participant-role">Group</span>
                                </a>
                            </div>
                        </template>
                        <template v-else>
                            <div
                                v-for="(member, index) in groupMembers"
                                :key="`group-members-${member.user_id}`"
                                class="flex-100 pad-5 group-member"
                            >
                                <a href="#" class="user-wrapper">
                                    <img
                                        :src="member.headshot"
                                        alt=""
                                        class="profile-img"
                                    />
                                    <span class="participant-name">
                                        {{ member.name }}
                                    </span>
                                    <input
                                        type="checkbox"
                                        class="participant-check"
                                        :id="member.user_id"
                                        :value="member.user_id"
                                        v-model="newGroupMembers"
                                        @click="checkBox($event, member.user_id)"
                                    />
                                </a>
                            </div>
                            <div class="text-right mar-t-10 flex-100">
                                <button
                                    href="#"
                                    disabled
                                    class="secondary-btn btn-de-default disabled-btn create-group-btn"
                                    @click="create_group_thread()"
                                >
                                    Create
                                </button>
                            </div>
                        </template>
                    </div>
                </div>

                <!-- <div class="modal-footer-padding">
                    <div class="d-flex justify-content-start">
                        <button class="btn btn-save-note" disabled>Send</button>
                        <button type="reset" class="hide resetForm">
                            Reset
                        </button>
                    </div>
                </div> -->
            </div>
        </div>
    </div>
</template>

<script>
import { gsap } from "gsap";
import Axios from "../../store/axios";
import store from "../../store";
import {mapGetters} from "vuex";

export default {
    name: "NewMessage",
    data() {
        return {
            searchQuery: null,
            selectedGroup: 0,
            newGroupMembers: [],
            groupMembersList: false,
            groupFullInfo: {},
            messengerUsers: {
                individuals: [],
                groups: []
            }
        };
    },
    created: function() {
        this.getConnections ('connections');
    },
    computed: {
        ... mapGetters(["memberLabel", "championLabel"]),
        individualsUsers(){
          if(this.searchQuery){
              return this.messengerUsers.individuals.filter((item)=>{
                return this.searchQuery.toLowerCase().split(' ').every(v => item.name.toLowerCase().includes(v))
              })
          }else{
            return this.messengerUsers.individuals;
          }
        },
        allgroups(){
          if(this.searchQuery){
              return this.messengerUsers.groups.filter((item)=>{
                return this.searchQuery.toLowerCase().split(' ').every(v => item.name.toLowerCase().includes(v))
              })
          }else{
                return this.messengerUsers.groups;
          }
        },
        groupMembers(){
          if(this.searchQuery){
              return this.groupFullInfo.members.filter((item)=>{
                return this.searchQuery.toLowerCase().split(' ').every(v => item.name.toLowerCase().includes(v))
              })
          }else{
            return this.groupFullInfo.members;
          }
        }

    },
    methods: {
        groupSelection(id,group_id) {
            this.groupMembersList = true;
            this.groupFullInfo = this.messengerUsers.groups[id];
            this.searchQuery = '';
            this.selectedGroup = group_id;
            this.newGroupMembers = [];
            setTimeout(() => {
                gsap.fromTo(
                    ".group-member",
                    { opacity: 0, y: 20 },
                    { opacity: 1, y: 0, duration: 0.275 }
                );
            }, 0);
        },
        checkBox(e,user_id) {

            const checkbox = $(e.target);
            const mainParent = checkbox.closest(".users-wrapper");
            if (
                checkbox
                    .closest(".users-wrapper")
                    .find(".participant-check")
                    .is(":checked")
            ) {
                mainParent
                    .find(".create-group-btn")
                    .prop("disabled", false)
                    .removeClass("disabled-btn");
                return;
            }
            mainParent
                .find(".create-group-btn")
                .prop("disabled", true)
                .addClass("disabled-btn");
        },

        back(e) {
            this.searchQuery = '';
            this.selectedGroup = 0;
        },
        create_group_thread() {
            this.showLoader ();
            var form = this.prepareData ();
            let url = "messengers/conversations/create";
            Axios.post ( url, form )
                .then ( ( res ) => {
                    const response = res.data.data.conversation;
                    // Socket Emit for new thread creation
                    console.log("create_group_thread")
                    console.log(response)
                    response.group_id = Number(response.group_id)
                    if (response.is_new_conversation) {
                        this.$socket.emit('create_new_thread', { data: response })
                    }

                    this.$emit ( 'updateConversationsList', response );
                    if(response.is_new_conversation === true) this.success ( 'New Conversation Created Successfully' );
                    $ ( '#createMsgModal' ).modal ( 'toggle' );
                    this.groupMembersList = false;
                    this.back('');
                    this.hideLoader ();
                } ).catch ( err => {
                this.error ( 'Something went wrong. Please try again later' );
                this.hideLoader ();
            } );
        },
        createNewThread(user_id) {
            this.showLoader ();
            let form = new FormData ();
            form.set ( 'users', user_id );
            let url = "messengers/conversations/create";
            Axios.post ( url, form )
                .then ( ( res ) => {
                    const response = res.data.data.conversation;
                    // Socket Emit for new thread creation
                    console.log("createNewThread")
                    console.log(response)
                    console.log(res.data.data.receiver_conversation)
                    if (response.is_new_conversation) {
                        this.$socket.emit('create_new_thread', { data: res.data.data.receiver_conversation })
                    }

                    this.$emit ( 'updateConversationsList', response );
                    if(response.is_new_conversation === true) this.success ( 'New Conversation Created Successfully' );
                    $ ( '#createMsgModal' ).modal ( 'toggle' );
                    this.hideLoader ();
                } ).catch ( err => {
                    if(err.response.data.message == 'Selected user permissions have been revoked.'){
                        this.error ( err.response.data.message );
                    }else{
                        this.error ( 'Something went wrong. Please try again later' );
                    }
                this.hideLoader ();
            } );
        },
        prepareData () {
            const form = new FormData ();
            if (this.selectedGroup != 0 && this.selectedGroup != undefined && this.selectedGroup != null) {
                form.set ( "group_id", this.selectedGroup );
            }
            form.set ( 'users', this.newGroupMembers.toString () );
            return form;
        },
        getConnections ( { commit }, type ) {
            let primaryUserId = store.getters.user.id;
            //let secondaryUserId = (store.getters.rolesCount == 2) ? store.getters.secondaryUser.id : 0 ;
            //console.log ( primaryUserId, secondaryUserId );
            //if ( store.getters.userRoles.cit !== undefined && store.getters.rolesCount == 1 ) window.location = '/connect';
            Axios.get ( 'messengers/users-and-groups' + '?primary_user_id=' + primaryUserId ).then ( response => {
                if ( ! ( response.data.status == 200 && response.data.message == "Connected and suggested matches list." ) ) return;
                let data = response.data;
                this.messengerUsers.individuals = [];
                this.messengerUsers.groups = [];
                if (data.data.connected_admins.length > 0) {
                    data.data.connected_admins.forEach(admin=>{
                        this.messengerUsers.individuals.push({
                            user_id: admin.id,
                            name: admin.full_name,
                            headshot:
                                admin.headshot,
                            role: "Admin"
                        });
                    })
                }

                if (data.data.connected_champions.length > 0) {
                    data.data.connected_champions.forEach(champion=>{
                        this.messengerUsers.individuals.push({
                            user_id: champion.id,
                            name: champion.full_name,
                            headshot:
                                champion.headshot,
                            role: this.championLabel
                        });
                    })
                }

                if (data.data.connected_members.length > 0) {
                    data.data.connected_members.forEach(member=>{
                        this.messengerUsers.individuals.push({
                            user_id: member.id,
                            name: member.full_name,
                            headshot:
                                member.headshot,
                            role: this.memberLabel
                        });
                    })
                }

                data.data.enrolled_groups.forEach(group=>{

                    var members = [];
                    var membersSummary = '';

                    if (group.enrolled_users.length <= 3) {
                        for (var i = 0; i < group.enrolled_users.length; i++) {
                            membersSummary += group.enrolled_users[i].full_name+', '
                        }
                    } else {
                        for (var i = 0; i < 3; i++) {
                            membersSummary += group.enrolled_users[i].full_name+', ';
                        }
                        membersSummary += ' and '+(group.enrolled_users.length - 3)+' others';

                    }
                    for (var i = 0; i < group.enrolled_users.length; i++) {
                        if(store.getters.user.id == group.enrolled_users[i].id){
                            continue;
                        }
                        members.push({
                            name: group.enrolled_users[i].full_name,
                            headshot:
                                group.enrolled_users[i].headshot,
                            role: "Admin",
                            user_id: group.enrolled_users[i].id
                        });
                    }
                    this.messengerUsers.groups.push({
                        id: group.id,
                        name: group.name,
                        membersSummary:
                            membersSummary,
                        headshot: group.icon,
                        members: members
                    });
                })
                // if (Object.keys(data.data.enrolled_groups).length > 0) {
                //     for (var key in data.data.enrolled_groups) {
                //         data.data.enrolled_groups[key].forEach(group=>{
                //             var membersSummary = group.user.first_name;
                //             var members = [];
                //             members.push({
                //                 name: group.user.name,
                //                 headshot:
                //                     group.user.profile_pic,
                //                 role: "Admin",
                //                 user_id: group.user.id
                //             });
                //             if (group.enrolled_users.length <= 3) {
                //                 for (var i = 0; i < group.enrolled_users.length; i++) {
                //                     membersSummary += ', '+group.enrolled_users[i].first_name
                //                 }
                //             } else {
                //                 for (var i = 0; i < 3; i++) {
                //                     membersSummary += ', '+group.enrolled_users[i].first_name;
                //                 }
                //                 membersSummary += ' and '+(group.enrolled_users.length - 3)+' others';

                //             }

                //             for (var i = 0; i < group.enrolled_users.length; i++) {
                //                 members.push({
                //                     name: group.enrolled_users[i].name,
                //                     headshot:
                //                         group.enrolled_users[i].profile_pic,
                //                     role: "Admin",
                //                     user_id: group.enrolled_users[i].id
                //                 });
                //             }
                //             this.messengerUsers.groups.push({
                //                 id: group.group_id,
                //                 name: group.name,
                //                 membersSummary:
                //                     membersSummary,
                //                 headshot: group.icon,
                //                 members: members
                //             });
                //         })
                //     }
                // }
            } ).catch ( err => {
                console.log ( err )
            } );
        },
        groupMessageName(name){
            if(name.length > 30) {
               return name.substring(0,30)+' ...';
            } else {
               return name;

            }
        },
    }
};
</script>

<style lang="scss" scoped>
@import "resources/assets/sass/abstracts/_variables.scss";

.user-wrapper {
    display: grid;
    grid-template-columns: auto 1fr auto;
    grid-template-rows: 1fr auto;
    align-items: center;
    background: color(white);
    transition: background ease 0.275s;

    .profile-img {
        grid-area: 1 / 1 / span 2 / span 1;
        margin-right: 10px;
    }

    .participant-name {
        font-family: $semiBold;
        font-size: 16px;
        color: color(dark);
    }

    .participant-role {
        padding: 0 6px;
        background: #2982bb;
        color: #ffffff;
        font-family: $bold;
        font-size: 14px;
        width: max-content;
        border-radius: 16px;
    }

    .participant-check {
        appearance: none;
        height: 25px;
        width: 25px;
        background: color(white);
        border: 1px solid color(secondary);
        border-radius: 50%;
        position: relative;
        cursor: pointer;

        &:after {
            content: "\e907";
            font-family: "icomoon";
            font-size: 10px;
            opacity: 0;
            transition: opacity 0.275s ease;
            color: color(white);
            position: absolute;
            left: 50%;
            transform: translate(-50%, -50%);
            top: calc(50% + 1px);
        }

        &:checked {
            background: color(secondary);

            &:after {
                opacity: 1;
            }
        }
    }

    .group-info {
        grid-area: 2 / 2;
        color: color(dark);
    }

    &:hover {
        background: darken(color(white), 4%);
    }
}

#createMsgModal .modal-dialog {
    max-width: 420px;
}

.rounded-img-group {
    border-radius: 50%;
    width: 45px;
    height: 45px;
    object-fit: cover;
}

.info-nav-bar {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    align-items: flex-start;

    .back-nav {
        width: max-content;
    }

    .group-info {
        display: flex;
        flex-direction: column;
        align-items: center;

        img {
            border-radius: 50%;
            object-fit: cover;
            height: 50px;
            width: 50px;
            margin-bottom: 8px;
        }

        h6 {
            font-size: 14px;
            font-family: $bold;
        }
    }
}
</style>
