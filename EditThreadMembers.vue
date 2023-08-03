<template>
    <div
        class="modal fade"
        id="editThreadMembers"
        role="dialog"
        aria-hidden="true"
    >
        <div class="modal-dialog taskEditor" role="document">
            <div class="modal-content">
                <div class=" modalEditor taskEditorModal">
                    <div class="d-flex justify-content-between align-center">
                        <h5 class="modal-title modalEditorTitle">
                            <span>Edit Group Members</span>
                        </h5>
                        <button
                            type="button"
                            class="close CloseBtn"
                            data-dismiss="modal"
                            aria-label="Close"
                            @click="groupMembersList = false"
                        >
                            <i class="icon-close"></i>
                        </button>
                    </div>
                </div>

                <div class="modal-body">
                    <div class="mar-b-10 info-nav-bar">
                        <div class="group-info" v-if="groupInfo">
                            <img :src="groupInfo.icon" alt="" />
                            <h6>{{ groupInfo.name }}</h6>
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

                        <div
                            v-for="(member, index) in groupMembersQuery"
                            :key="`group-members-${index}`"
                            class="flex-100 pad-5 group-member" v-if="member.id != user.id"
                        >
                            <a href="#" class="user-wrapper">
                                <img
                                    :src="member.headshot"
                                    alt=""
                                    class="profile-img"
                                />
                                <span class="participant-name">
                                    {{ member.full_name }}
                                </span>
                                <input
                                    type="checkbox"
                                    class="participant-check"
                                    :id="member.id"
                                    :value="member.id"
                                    v-model="selectedMembers"
                                />
                            </a>
                        </div>
                        <div class="text-right mar-t-10 flex-100">
                            <button
                                href="#"
                                class="secondary-btn btn-de-default create-group-btn"
                                :class="{ 'disabled-btn': selectedMembers.length == 0 }"
                                @click="updateGroupThread()"
                            >
                                Update
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>

import Axios from "../../store/axios";
import {mapGetters} from "vuex";

export default {
    name: "EditThreadMembers",
    props : [ "groupMembers", "threadMembersIds", "conversationId", "groupInfo" ],
    data() {
        return {
            searchQuery: null,
            groupMembersList: false,
            selectedMembers: []
        };
    },
    created: function() {},
    computed: {
        ... mapGetters ( [
            "user"
        ] ),
        groupMembersQuery(){
            if(this.searchQuery){
                return this.groupMembers.filter((item)=>{
                    return this.searchQuery.toLowerCase().split(' ').every(v => item.full_name.toLowerCase().includes(v))
                });
            }else{
                return this.groupMembers;
            }
        },
    },
    watch : {
        threadMembersIds : function ( newVal, oldVal ) {
            this.selectedMembers = newVal;
        },
    },
    methods: {
        updateGroupThread() {
            this.showLoader ();
            let form = this.prepareData();
            let url = "messengers/conversations/"+this.conversationId+"/update/members";
            Axios.post ( url, form )
                .then ( ( res ) => {
                    const response = res.data.data.conversation;
                    response.update_list = true;
                    this.$socket.emit('edit_thread_members', response)
                    this.$emit ( 'updateConversationsList', response );
                    $ ( '#editThreadMembers' ).modal ( 'toggle' );
                    this.hideLoader ();
                } ).catch ( err => {
                this.error ( 'Something went wrong. Please try again later' );
                this.hideLoader ();
            } );
        },
        prepareData () {
            const form = new FormData ();
            form.set ( 'users', this.selectedMembers.toString () );
            return form;
        },
        /*selectMember(userId){
            if (this.selectedMembers.includes(userId)) {
                let index = this.selectedMembers.indexOf(userId);
                this.selectedMembers.splice(index, 1);
            } else {
                this.selectedMembers.push(userId)
            }
        }*/
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
    display: flex;
    justify-content: center;

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
