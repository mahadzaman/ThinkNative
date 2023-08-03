<template>
    <section class="container about-group-wrapper">
        <div class="description-section">
            <h5>About</h5>
            <p class="group-description mar-t-20">
                {{ groupData.description }}
            </p>
        </div>
        <div class="tags-section mar-t-30">
            <h5>Tags</h5>
            <div
                v-if="groupData.tags && groupData.tags.length > 0"
                class="tags-container mar-t-20"
            >
                <span v-for="tag in groupData.tags" class="tag">{{
                    tag.name
                }}</span>
            </div>
        </div>
        <div class="members-section mar-t-30">
            <div class="members-header flex-box space-between align-center">
                <div class="flex-col-50 raw-flex align-center">
                    <h5>Members</h5>
                    <span
                        v-if="groupDataMembers"
                        class="about-members-count badge"
                        >{{
                            groupDataMembers.length +
                                (groupOwner == null ? 0 : 1)
                        }}</span
                    >
                </div>
                <div class="search-container flex-col-40">
                    <input
                        v-model="searchMember"
                        type="text"
                        placeholder="Search Name"
                        @keydown.enter="searchGroupMember"
                        @keydown.188="searchGroupMember"
                    />
                    <button class="btn-de-default" @click="searchGroupMember">
                        <i class="icon-search"></i>
                    </button>
                </div>
            </div>
            <div class="members-container flex-box mar-t-20">
                <div v-if="groupOwner" class="flex-col-25">
                    <div
                        class="member-tile custom-upload"
                        @click="
                            viewMemberProfile(groupOwner.id, 'not_required')
                        "
                    >
                        <div class="circular-img">
                            <img :src="getGroupOwnerProfilePic()" />
                        </div>
                        <span class="member-name">{{
                            groupOwner.full_name
                        }}</span>
                        <span class="un-emph">Organizer</span>
                    </div>
                </div>
                <div
                    v-if="groupDataMembers"
                    v-for="groupMember in groupDataMembers"
                    class="flex-col-25"
                >
                    <div
                        class="member-tile custom-upload"
                        @click="
                            viewMemberProfile(
                                groupMember.id,
                                'not_required'
                            )
                        "
                    >
                        <div class="circular-img">
                            <img :src="groupMember.headshot" />
                        </div>
                        <span class="member-name">{{
                            groupMember.full_name
                        }}</span>
                    </div>
                </div>
            </div>
        </div>
    </section>
</template>

<script>
import { mapActions } from "vuex";
import Axios from "../../store/axios";

export default {
    name: "aboutGroup",
    props: ["groupData", "groupOwner", "groupDataMembers"],
    data() {
        return {
            s3Url: "https://thinknative.s3-us-west-2.amazonaws.com/",
            searchMember: ""
        };
    },
    methods: {
        ...mapActions(["getGroupWallData"]),
        getGroupOwnerProfilePic: function() {
            return this.groupOwner.profile_pic
                ? this.groupOwner.profile_pic
                : "";
        },
        searchGroupMember: function() {
            this.$emit("searchGroupMembers", this.searchMember);
        },
        viewMemberProfile: function(userId, roleId) {
            this.$emit("viewMemberProfile", userId, roleId);
        }
    }
};
</script>

<style lang="scss" scoped>
@import "resources/assets/sass/abstracts/_variables.scss";

.description-section,
.tags-section,
.members-section {
    margin-bottom: 30px;

    h5 {
        color: color(secondary);
        font-family: $bold;
        margin-bottom: 0;
    }
}

.members-section,
.tags-section,
.description-section {
    @include tabletZone {
        padding: 0 25px;
    }

    @include tablet {
        padding: 0 15px;
    }
}

.members-header {
    & > .flex-col-50 {
        padding-left: 0;
    }

    & > .flex-col-40 {
        @include tablet {
            flex: 0 0 40%;
        }

        @include mobile {
            flex: 0 0 100%;
            margin-top: 15px;
        }
    }
}

.description-section {
    p {
        color: color(dark);
        margin-bottom: 0;
    }
}

.tags-section {
    .tags-container {
        display: flex;
        flex-wrap: wrap;
        margin-left: -5px;

        .tag {
            background-color: color(secondary);
            margin-left: 5px;
            margin-top: 5px;
            padding: 10px 15px;
            white-space: nowrap;
            border-radius: 25px;
            color: color(white);
        }
    }
}

.members-container {
    margin-left: -25px;
    margin-right: -25px;

    @include mobile {
        margin-left: -15px;
        margin-right: -15px;
    }

    .flex-col-25 {
        margin-bottom: 25px;
        padding: 0 25px;

        @include mobile {
            padding: 0 15px;
        }

        @include tablet {
            flex: 0 0 33.33%;
        }

        @include mobile {
            flex: 0 0 50%;
        }
    }
    .member-tile {
        display: flex;
        flex-direction: column;
        color: color(newLight);
        align-items: center;
        padding: 20px 10px;
        background: color(white);
        height: 100%;
        box-shadow: $card-shadow;
        border-radius: $border-radius;
        position: relative;

        .circular-img {
            width: 80px;
            height: 80px;
            border-radius: 50%;
            background: color(lighter);
            box-shadow: $card-shadow;
            position: relative;
            overflow: hidden;

            img {
                min-width: 100%;
                min-height: 100%;
                @include centralize-absolute;
                object-fit: cover;
            }

            &:after {
                content: "";
                width: 100%;
                height: 100%;
                border-radius: 50%;
                border: 2px solid color(secondary);
                position: absolute;
                top: 0;
                left: 0;
            }
        }

        span {
            &:not(.un-emph) {
                margin-top: 5px;
                font-family: $semiBold;
            }

            &.un-emph {
                position: absolute;
                bottom: 3px;
            }
        }
    }
}

.search-container {
    position: relative;

    input[type="text"] {
        width: 100%;
        border: 3px solid color(bg-main);
        padding: 8px 18px;
        border-radius: 50px;
        outline: none;
        background-color: color(white);
        color: color(dark);

        &:focus {
            color: color(dark);
        }

        &::placeholder {
            color: color(dark);
        }
    }

    button {
        color: color(secondary);
        font-size: 20px;
        position: absolute;
        right: 18px;
        top: 8px;
    }
}

.about-members-count.badge {
    background: color(secondary);
    color: color(white);
    margin-left: 10px;
    border-radius: 50%;
    padding: 5px;
    font-size: 12px;
    min-height: 30px;
    min-width: 30px;
    @include centralize-flex;
}
</style>
