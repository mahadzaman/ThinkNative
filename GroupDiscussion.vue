<template>
    <div class = "flex-box">
        <div class = "wall-col mar-0-auto">
            <div class = "scrollMainSec">
                <div class = "create-post-wrapper">
                    <div class = "post-box">
                        <div class = "create-session-container">
                            <form method = "post" action = "" class = "postForm" id = "postForm"
                                  @submit.prevent = "createPostForm">
                                <div class = "form-group mar-b-0 raw-flex">
                                    <div class = "profilePic">
                                        <img :src = "user.headshot" />
                                    </div>
                                    <div class = "create-post-text form-control">
                                        <span name = "postBody" contenteditable = "true" @input = "onInput"
                                              @paste = "pasteContent"
                                              @cut = "cutEvent"
                                              data-text = "What's on your mind?" class = "editable form-control"></span>
                                    </div>

                                </div>
                                <div v-if = "postFormErrors.postBody"
                                     class = "ml-5 mb-0 raw-flex align-center vee-validate">
                                    <span class = "ml-4 has-error-span">{{ postFormErrors.postBody }}</span>
                                </div>
                                <div class = "post-img-placeholder hide post-media-container">
                                    <img src = "" alt = "">
                                    <a href = "javascript:;" class = "removeMedia gray-primary-btn norm-btn"
                                       v-on:click = "removeMedia">
                                        <i class = "icon-close"></i>
                                    </a>
                                </div>
                                <div class = "post-vid-placeholder hide post-media-container">
                                    <video controls muted>
                                        <source src = "" />
                                    </video>
                                    <a href = "javascript:;" class = "removeMedia gray-primary-btn norm-btn"
                                       v-on:click = "removeMedia">
                                        <i class = "icon-close"></i>
                                    </a>
                                </div>

                                <div class = "post-media-actions-wrapper">
                                    <div class = "custom-input-file-container mar-r-30">
                                        <span class = "custom-upload group-post-media-btn tooltip-c" tooltip-position="top" v-on:click = "postMedia">
                                            <i class = "icon-picture mar-r-5"></i>
                                            Photo/Video
                                            <span class="tooltip-text">Max. 180MB<br><br>Note: MOV files require additional time to upload.</span>
                                        </span>
                                        <input name = "postMedia" type = "file" class = "postMedia"
                                               v-on:change = "mediaPopulate"
                                               accept = "image/*,video/mp4,video/x-m4v,video/*" />
                                    </div>
                                </div>
                                <div class = "text-right">
                                    <button type = "submit" class = "primary-btn btn-de-default mar-l-auto">Post
                                    </button>
                                    <button type = "reset" class = "hide resetForm">Reset</button>
                                </div>
                            </form>
                        </div>
                    </div>
                </div>
                <div v-if = "group_data.group_posts" v-for = "(groupPost, index) in group_data.group_posts"
                     :key = "groupPost.post.id" :class = "'mainSec post-' + groupPost.post.id " :id = "groupPost.post.id">
                    <div class = "flex-start">
                        <div class = "profilePic">
                            <img :src = "getPostOwnerProfilePicture(groupPost.owner.profile_pic)" />
                        </div>
                        <div class = "profileNameSec">
                            <h3 class = "pName custom-upload" @click="viewMemberProfile(groupPost.owner.id, 'not_required')">{{ groupPost.owner.full_name }}</h3>
                            <p class = "timeProfile">{{ groupPost.post.type }}</p>
                        </div>
                        <div v-if = "user.id == groupPost.post.user_id"
                             class = "dropdown show removeArrowDown imgDots ">
                            <a class = "btn btn-secondary dropdown-toggle" href = "javascript:;"
                               role = "button" id = "dropdownMenuLink" data-toggle = "dropdown"
                               aria-haspopup = "true" aria-expanded = "false">
                                <img src = "/img/profileTask/dotsGoals.png" />
                            </a>

                            <div class = "dropdown-menu" aria-labelledby = "dropdownMenuLink">
                                <a class = "dropdown-item" href = "javascript:;"
                                   v-on:click = "removePost(groupPost.post.id)">
                                    <i class = "fa fa-trash mr-1"></i>Delete
                                </a>
                                <a class = "dropdown-item" href = "javascript:;" data-toggle = "modal"
                                   data-backdrop = "static" data-keyboard = "false"
                                   data-target = "#postModal" v-on:click = "editPost(groupPost.post)">
                                    <i class = "fa fa-pencil-square-o mr-1" aria-hidden = "true"></i>Edit
                                </a>
                            </div>
                        </div>
                    </div>
                    <p v-if = "groupPost.post.body" class = "postHeading" v-html = "replaceNbsp(groupPost.post.body)"></p>
                    <div v-if = "groupPost.post.attachment_type" class = "postPic">
                        <a href="javascript:void(0)" @click="openImgPopup($event)">
                            <img v-if = "isImageAttachment(groupPost.post.attachment_type)"
                                :src = "groupPost.post.attachment_url" />
                        </a>
                        <video v-if = "isVideoAttachment(groupPost.post.attachment_type)"
                               class = "video-js vjs-big-play-centered vjs-16-9" data-setup = '{"fluid": true}' controls
                               preload = "auto" width = "425" height = "264"
                               :poster = "getMediaThumbnail(groupPost.post.media.thumbnail)">
                            >
                            <source :src = "groupPost.post.attachment_url" type = "video/mp4" />
                            <p class = "vjs-no-js">
                                To view this video please enable JavaScript, and
                                consider upgrading to a web browser that
                                <a href = "https://videojs.com/html5-video-support/" target = "_blank">supports HTML5
                                                                                                       video</a>
                            </p>
                        </video>
                    </div>
                    <div class = "flex-start paddingcmt">
                        <div class = "flex-start">
                            <div class = "likeCount">
                                <p>{{ groupPost.all_comments }}</p>
                            </div>
                            <div class = "likeP">
                                <p v-if = "groupPost.all_comments == 1">Comment</p>
                                <p v-else>Comments</p>
                            </div>
                        </div>
                    </div>
                    <div class = "opinions-wrapper" :data-id = "groupPost.post.id"
                         v-bind:class = "{'view-less': groupPost.all_comments > 3}"
                         v-if = "groupPost.all_comments > 0">
                        <div v-for = "comment in groupPost.post.comments" class = "opinion-box"
                             :class = "'comment-box-' + comment.id " v-if="comment.commentator !== null">
                            <div class = "opinion-header-wrapper">
                                <div class = "opinion-author">
                                    <a href = "javascript:;">
                                        <img :src = "getCommentatorImage(comment.commentator.profile_pic)" />
                                    </a>
                                </div>
                                <div class = "opinion">
                                    <!--Role id is not required so we have to identify all places where we are using viewMemberProfile function-->
                                    <a class = "commenter-name custom-upload" href = "javascript:;" @click="viewMemberProfile(comment.commentator.id, 'not_required')">
                                        {{ comment.commentator.full_name }}&nbsp;</a>
                                    <p class = "comment-body" v-html = "comment.body"></p>
                                    <div v-if = "comment.media != null" class = "post-media-container">
                                        <a @click.prevent="openImgPopup($event)" href = "javascript:void(0)" target = "_blank"
                                           :data-title = "comment.body">
                                            <img class = "opinion" :src = "comment.attachment_url" />
                                        </a>
                                    </div>
                                    <div class = "opinion-actions">
                                        <a v-if = "comment.replies.length > 0" href = "javascript:;"
                                           class = "replies-btn" v-on:click = "toggleComments">
                                            <span>{{ comment.replies.length }}</span>&nbsp;Replies</a>
                                        <a href = "javascript:void(0)" v-on:click = "replyOnComment"
                                           :data-id = "'comment_id'+comment.id"
                                           :data-parent_id = "comment.id"
                                           :data-post_id = "groupPost.post.id" class = "reply-btn">Reply</a>
                                        <a href = "javascript:void(0)" v-on:click = "cancelReply($event)"
                                           :data-id = "'comment_id'+comment.id" class = "cancel-opinion hide">Cancel</a>
                                    </div>

                                    <div class = "reply-box-wrapper hide">
                                        <span contenteditable = "true"
                                              v-on:keypress = "onAddingComment($event, comment)"
                                              v-on:keydown = "removeCommentMedia"
                                              @paste = "pasteContent"
                                              @cut = "cutEvent"
                                              :data-post-index = "index" :data-parent_id = "comment.parent_id"
                                              class = "editable comment-box form-control"></span>
                                        <div class = "custom-input-file-container">
                                            <span @click="commentUpload" class = "custom-upload"><i class = "icon-photo-camera"></i></span>
                                            <input @change="replyImage" type = "file" class = "replyImage" accept = "image/*" />
                                        </div>
                                    </div>
                                </div>
                                <div class = "opinion-actions">
                                    <a href = "javascript:;" class = "post-actions-toggle"
                                       v-if = "user.id == comment.commentator.id"
                                       v-on:click = "postActionToggle">
                                        <i class = "icon-dots"></i>
                                    </a>
                                    <div class = "post-actions-list">
                                        <a v-if = "user.id == comment.commentator.id" v-on:click = "editComment"
                                           href = "javascript:;" :data-id = "'comment_id'+comment.id"
                                           :data-post_id = "groupPost.post.id"
                                           :data-text = "comment.body" class = "edit-opinion">
                                            <i class = "fa fa-pencil-square-o"></i>
                                            <span>Edit</span>
                                        </a>
                                        <a v-if = "user.id == comment.commentator.id" href = "javascript:;"
                                           :data-id = "'comment_id'+comment.id" v-on:click = "removeComment"
                                           :data-post_id = "groupPost.post.id" :data-post-index = "index"
                                           class = "del-opinion">
                                            <i class = "fa fa-trash"></i>
                                            <span>Delete</span>
                                        </a>
                                    </div>
                                </div>
                            </div>
                            <div v-for = "reply in comment.replies" class = "replies-wrapper hide"
                                 :class = "'comment-box-' + reply.id ">
                                <GroupReplies :reply = "reply" :user_id = "user.id" :postKey = "index"
                                              :group_id = "group_data.group.id" :post_id = "groupPost.post.id" @viewMemberProfile = "viewMemberProfile" @repImgLink="triggerFromReplies"/>
                            </div>
                        </div>
                        <div class = "view-all-comments view-actions">
                            <a href = "javascript:;" v-on:click = "viewAllComments">View All Comments</a>
                        </div>
                        <div class = "view-less-comments view-actions hide">
                            <a href = "javascript:;" v-on:click = "viewLessComments">View Less Comments</a>
                        </div>
                    </div>

                    <div class = "opinions-footer mt-3">
                        <div class = "opinion-author">
                            <a href = "javascript:;">
                                <img :src = "user.headshot" alt = "">
                            </a>
                        </div>
                        <div class = "reply-box-wrapper">

                            <span :data-post_id = "groupPost.post.id" data-text = "Write a Comment"
                                  :data-post-index = "index" v-on:keypress = "onAddingComment($event, {})"
                                  v-on:keydown = "removeCommentMedia"
                                  @paste = "pasteContent"
                                  @cut = "cutEvent"
                                  contenteditable = "true" class = "editable comment-box form-control"></span>
                            <div class = "custom-input-file-container">
                                <span v-on:click = "commentUpload" class = "custom-upload"><i
                                    class = "icon-photo-camera"></i></span>
                                <input type = "file" v-on:change = "replyImage" class = "replyImage"
                                       accept = "image/*" />
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Post Modal -->
        <div class = "modal fade" id = "postModal" tabindex = "-1" role = "dialog" aria-hidden = "true">
            <div class = "modal-dialog taskEditor" role = "document">
                <div class = "modal-content">
                    <div class = " modalEditor taskEditorModal">
                        <div class = "d-flex justify-content-between align-center">
                            <h5 class = "modal-title">
                                <span style = "color: #FFFFFF;">Update Post</span>
                            </h5>
                            <button type = "button" class = "close CloseBtn" data-dismiss = "modal" aria-label = "Close"
                                    v-on:click = "resetFormData">
                                <i class = "icon-close"></i>
                            </button>
                        </div>
                    </div>
                    <form method = "post" action = "" class = "postForm" id = "updatePostForm"
                          @submit.prevent = "createPostForm">
                        <div class = "modal-body noteEditorBodyPadding">
                            <div class = "form-group mar-b-0 raw-flex">
                                <div class = "profilePic">
                                    <img :src = "user.profile_pic" />
                                </div>
                                <div class = "create-post-text form-control">
                                        <span name = "ePostBody" contenteditable = "true" @input = "onInput"
                                              @paste = "pasteContent"
                                              @cut = "cutEvent"
                                              data-text = "What's on your mind?" class = "editable form-control"></span>
                                </div>

                            </div>
                            <div v-if = "postFormErrors.postBody"
                                 class = "ml-5 mb-0 raw-flex align-center vee-validate">
                                <span class = "ml-4 has-error-span">{{ postFormErrors.postBody }}</span>
                            </div>
                            <div class = "post-img-placeholder hide post-media-container">
                                <img src = "" alt = "">
                                <a href = "javascript:;" class = "removeMedia gray-primary-btn norm-btn"
                                   v-on:click = "removeMedia">
                                    <i class = "icon-close"></i>
                                </a>
                            </div>
                            <div class = "post-vid-placeholder hide post-media-container">
                                <video controls muted>
                                    <source src = "" />
                                </video>
                                <a href = "javascript:;" class = "removeMedia gray-primary-btn norm-btn"
                                   v-on:click = "removeMedia">
                                    <i class = "icon-close"></i>
                                </a>
                            </div>
                            <div class = "post-media-actions-wrapper">
                                <div class = "custom-input-file-container mar-r-30">
                                        <span class = "custom-upload group-post-media-btn" v-on:click = "postMedia">
                                            <i class = "icon-picture mar-r-5"></i>
                                            Photo/Video
                                        </span>
                                    <input name = "postMedia" type = "file" class = "postMedia"
                                           v-on:change = "mediaPopulate"
                                           accept = "image/*,video/mp4,video/x-m4v,video/*" />
                                </div>
                            </div>
                        </div>
                        <div class = "modal-footer-padding">
                            <div class = "d-flex justify-content-start">
                                <input id = "postMediaRemove" name = "postMediaRemove" type = "hidden" value = "0" />
                                <button type = "submit" class = "primary-btn btn-de-default mar-l-auto">Post</button>
                                <button type = "reset" class = "hide resetForm">Reset</button>
                            </div>
                        </div>
                    </form>
                </div>
            </div>
        </div>
        <Image-Viewer @click="closePopup" :img-popup-open="imgViewerOpen" :src-link="imageViewerLink" />
        <!-- Post Modal -->
    </div>
</template>

<script>

    import { mapGetters } from "vuex";
    import axios from "../../store/axios";
    import GroupReplies from "./GroupReplies";
    import ImageViewer from "./WallImgModal"

    export default {
        name : 'groupDiscussion',
        components : { GroupReplies, ImageViewer },
        component : {
            GroupReplies
        },
        props : [ "groupOwnerData", "authUser" ],
        data () {
            return {
                supportedImages : [ 'jpg', 'jpeg', 'jfif', 'gif', 'png' ],
                postFormErrors : [],
                postBody : null,
                postFile : null,
                updateFlag : 0,
                postMediaRemove : 0,
                id : "",
                imageViewerLink: '',
                imgViewerOpen: false

            };
        },
        computed : {
            ... mapGetters ( [ "user", "group_data" ] )
        },
        created () {
            $ ( document ).on ( 'click', function () {
                $ ( ".post-actions-list" ).each ( function () {
                    if ( $ ( this ).css ( 'visibility' ) == 'visible' ) {
                        $ ( this ).css ( {
                            'visibility' : "",
                            'opacity' : "",
                            "transform" : ""
                        } );
                    }
                } );
            } );
            this.fetch_tagging_users ();
        },
        updated : function () {
            //------------DO NOT REMOVE THIS CODE---------------
            // this.hideLoader()
            //
            // this.$nextTick(function () {
            //     let elem , activity_id
            //     activity_id = this.$router.currentRoute.query.post_id;
            //
            //     if(this.$router.currentRoute.hash == '#discussion' && activity_id) {
            //         elem = this.$el.getElementsByClassName("post-" + activity_id)[0];
            //         if (elem) {
            //             this.scrollToElementAndHighlight(elem);
            //             let query = Object.assign({}, this.$route.query);
            //             console.log(query);
            //             delete query.post_id;
            //             this.$router.replace({ query });
            //             this.$router.push({ hash: 'discussion' });
            //         }
            //     }
            // })
        },
        watch : {
            group_data () {
                console.log ( this.group_data )
            }
        },
        methods : {
            triggerFromReplies (link) {
                // console.log(link);
                $('html, body').css('overflow', 'hidden');
                this.imageViewerLink = link;
                this.imgViewerOpen = true;
            },
            closePopup() {
                this.imgViewerOpen = false;
                $('html,body').css('overflow', '');
            },
            openImgPopup(e){
                $('html, body').css('overflow', 'hidden');
                let link = $(e.target).attr('src');
                this.imageViewerLink = link;
                this.imgViewerOpen = true;
            },
            viewAllComments : function ( e ) {
                $ ( e.target ).parent ().parent ().removeClass ( "view-less" );
                $ ( e.target ).parent ().siblings ( ".view-less-comments" ).removeClass ( "hide" );
            },
            viewLessComments : function ( e ) {
                $ ( e.target ).parent ().parent ().addClass ( "view-less" );
                $ ( e.target ).parent ().addClass ( "hide" );
            },
            getPostOwnerProfilePicture : function ( ownerPic ) {
                return ( ownerPic ) ? ownerPic : this.group_data.group.default_profile_pic;
            },
            isImageAttachment : function ( type ) {
                return this.supportedImages.includes ( type );
            },
            isVideoAttachment : function ( type ) {
                return ! this.supportedImages.includes ( type );
            },
            getMediaThumbnail : function ( thumbnail ) {
                return thumbnail ? thumbnail : '';
            },
            getCommentatorImage : function ( img ) {
                return ( img ) ? img : '/img/profile-pic.jpeg';
            },
            postMedia : function ( event ) {
                $ ( event.target ).siblings ( '.postMedia' ).click ();
            },
            mediaPopulate : function ( event ) {
                if ( ! ( event.target.files && event.target.files[ 0 ] ) ) return;
                this.showLoader();
                let response = this.postImage ( event.target );
                console.log(response , '---')
                let input = $ ( event.target );
                input.parent ().parent ().addClass ( "hide" );
                let type = input[ 0 ].files[ 0 ].type;
                new Promise ( ( resolve, reject ) => {
                    resolve ( type );
                } );
                // this.hideLoader();
                $ ( "#postMediaVideo" ).attr ( "src", "" ).addClass ( "hide" );
                $ ( "#postMediaImage" ).attr ( "src", "" ).addClass ( "hide" );
                this.postFile = event.target.files[ 0 ];
            },
            postImage ( input ) {
                let self = this;
                const maxSize = 180; // Max file size Allowed
                let reader = new FileReader ();
                reader.onload = function ( e ) {
                    let size = input.files[ 0 ].size / 1048576;
                    size = size.toFixed ( 2 );
                    if ( size > maxSize ) {
                        self.error ( "File Size Too Large!, Given file size is " + size + " mb, files above 180 mb are not allowed" );
                        $(input).parent ().parent ().removeClass('hide');
                        $ ( "#postMediaVideo" ) .removeClass('hide');
                        $ ( "#postMediaImage" ) .removeClass('hide');
                        self.hideLoader();
                        return false;
                    } else {
                        self.hideLoader();
                    }
                    const validImageTypes = [ "image/gif", "image/jpeg", "image/png", "image/jfif", "image/jpg" ];
                    let placHolderImg = $ ( input ).parent ().parent ().siblings ( ".post-img-placeholder" );
                    let placHolderVid = $ ( input ).parent ().parent ().siblings ( ".post-vid-placeholder" );
                    if ( validImageTypes.includes ( input.files[ 0 ].type ) ) {
                        placHolderImg.find ( "img" ).attr ( "src", "" );
                        placHolderImg.find ( "img" ).attr ( "src", e.target.result );
                        placHolderImg.removeClass ( "hide" );
                        placHolderVid.addClass ( "hide" );
                    } else {
                        let source = placHolderVid.find ( "source" );
                        let vid = placHolderVid.children ( "video" );
                        source.attr ( "src", "" );
                        source[ 0 ].src = URL.createObjectURL ( input.files[ 0 ] );
                        vid[ 0 ].load ();
                        placHolderVid.removeClass ( "hide" );
                        placHolderImg.addClass ( "hide" );
                    }
                };
                reader.readAsDataURL ( input.files[ 0 ] );
            },
            removeMedia : function ( event ) {
                let parent = $ ( event.target ).parent ().parent ();
                if ( $ ( event.target ).hasClass ( 'icon-close' ) ) parent = $ ( event.target ).parent ().parent ().parent ();
                let placeHolders = parent.find ( ".post-img-placeholder, .post-vid-placeholder, .post-iframe-holder" );
                let inputFile = parent.find ( ".custom-input-file-container input[type='file']" );
                placeHolders.addClass ( "hide" );
                placeHolders.find ( "source, img, iframe" ).attr ( "src", "" );
                this.postMediaRemove = 1;
                inputFile.val ( "" );
                parent.find ( ".videoUrlHf" ).val ( "" );
                parent.find ( ".thumbnailHf" ).val ( "" );
                parent.find ( ".post-media-actions-wrapper" ).removeClass ( "hide" );
            },
            resetFormData () {
                this.postFile = null;
                this.postBody = null;
                // this.updateFlag = 0;

                $ ( '[name="postBody"]' ).text ( '' );
                $ ( '.resetForm' ).click ();
                if ( this.updateFlag == 1 && $ ( '.post-img-placeholder' ).hasClass ( 'hide' ) )
                    this.resetFormMedia ( $ ( '#updatePostForm .modal-body .post-vid-placeholder .removeMedia' ) );

                else if ( this.updateFlag == 1 && $ ( '.post-vid-placeholder' ).hasClass ( 'hide' ) )
                    this.resetFormMedia ( $ ( '#updatePostForm .modal-body .post-img-placeholder .removeMedia' ) );

                else if ( this.updateFlag == 0 && $ ( '.post-vid-placeholder' ).hasClass ( 'hide' ) )
                    this.resetFormMedia ( $ ( '#postForm .post-img-placeholder .removeMedia' ) );

                else if ( this.updateFlag == 0 && $ ( '.post-img-placeholder' ).hasClass ( 'hide' ) )
                    this.resetFormMedia ( $ ( '#postForm .post-vid-placeholder .removeMedia' ) );

                this.updateFlag = 0;
            },
            resetFormMedia ( elem ) {
                let parent = $ ( elem ).parent ().parent ();
                let placeHolders = parent.find ( ".post-img-placeholder, .post-vid-placeholder, .post-iframe-holder" );
                let inputFile = parent.find ( ".custom-input-file-container input[type='file']" );

                placeHolders.addClass ( "hide" );
                placeHolders.find ( "source, img, iframe" ).attr ( "src", "" );
                this.postMediaRemove = 0;
                inputFile.val ( "" );
                parent.find ( ".videoUrlHf" ).val ( "" );
                parent.find ( ".thumbnailHf" ).val ( "" );
                parent.find ( ".post-media-actions-wrapper" ).removeClass ( "hide" );
            },
            validatePost () {
                if ( this.updateFlag === 1 ) {
                    if ( $ ( '[name="ePostBody"]' ).html () == '<br>' || $ ( '[name="ePostBody"]' ).html () == '' )
                        this.postBody = null;
                    if ( $ ( '[name="ePostBody"]' ).text () == '' )
                        this.postBody = null;
                }
                if ( this.updateFlag === 0 ) {
                    if ( $ ( '[name="postBody"]' ).html () == '<br>' || $ ( '[name="postBody"]' ).html () == '' )
                        this.postBody = null;
                }
            },
            createPostForm : function ( e ) {
                this.postFormErrors = [];
                this.validatePost ();
                if ( ! this.postBody ) this.postFormErrors[ 'postBody' ] = "Please write some description of post";
                if ( ! this.postBody ) return false;

                this.showLoader ();
                const form = this.prepareData ( $ ( e.target ) );
                let url = 'groups/posts';
                if ( this.updateFlag === 1 ) url = 'groups/posts/' + this.id;
                axios.post ( url, form ).then ( ( res ) => {
                    if (typeof res == 'undefined') {
                        this.error ( 'Session Expired.' );
                        return;
                    }
                    const response = res.data.data;
                    this.postMediaRemove = 0;
                    if ( this.updateFlag === 1 ) {
                        this.getGroupPosts (this.id);
                        this.success ( 'Post Updated Successfully' );
                        $ ( '#postModal' ).modal ( 'toggle' );
                    } else {
                        this.getGroupPosts ();
                        this.success ( 'Post Added Successfully' );
                    }
                    this.resetFormData ();

                } ).catch ( (err) => {
                    this.error ( 'Something went wrong. Please try again later' );
                    this.hideLoader ();
                } );

            },
            onInput ( e ) {
                // this.postBody = e.target.innerHTML;
                this.postBody = e.target.innerHTML.replace(/style="[^"]*"/, "");
            },
            pasteContent ( e ) {
                let url = e.clipboardData.getData ( "text" ), elem = $ ( e.target );
                if ( $ ( elem ).hasClass ( 'editable' ) == false ) elem = $ ( elem ).closest ( '.editable.form-control' );
                let link = this.$helpers.validateURL ( url );
                if ( ! link ) return;

                if ( $ ( elem ).find ( '.meta-post' ).find ( '.meta-anchor' ).length > 0 ) return;
                elem.siblings ( ".meta-anchor" ).children ().html ( "" );
                this.$helpers.getMeta ( this, elem, link, false );
            },
            cutEvent ( e ) {
                let url = e.clipboardData.getData ( "text" ), elem = $ ( e.target );
                if ( $ ( elem ).hasClass ( 'editable' ) == false ) elem = $ ( elem ).parent ();
                let link = this.$helpers.validateURL ( url );

                if ( ! link || e.target.innerText == "" || e.target.innerText === "" ||
                    e.target.innerText == null || e.target.innerText === null ) {
                    elem.html ( "" );
                    $ ( ".create-post-text span" ).css ( "margin-bottom", "unset" ).css ( "min-height", "unset" );
                } else {
                    return;
                }
            },
            prepareData ( elem ) {
                if ( this.updateFlag === 1 ) this.postBody = $ ( elem ).find ( '[name="ePostBody"]' ).html ();
                if ( this.updateFlag === 0 ) this.postBody = $ ( elem ).find ( '[name="postBody"]' ).html ();
                
                this.postBody = this.postBody.replace(/style="[^"]*"/, "");
                let mentions = $(elem).find('[name="postBody"]').find(".atwho-inserted");
                let users = [];
                $.each(mentions, function (i, item) {
                    users.push($(item).attr("data-id"));
                });
                const form = new FormData ();
                form.set ( "users", users );
                form.set ( "video_url", '' );
                form.set ( "thumbnail", '' );
                form.set ( "postBody", this.postBody );
                form.set ( "group_id", this.group_data.group.id );
                if ( this.postFile != null ) form.set ( "postMedia", this.postFile );
                if ( this.updateFlag === 1 && this.postMediaRemove == 1 ) form.set ( "postMediaRemove", "1" )
                if ( this.updateFlag === 1 ) form.set ( "_method", "PUT" )
                return form;
            },
            getGroupPosts : function (postId) {
                axios.get ( 'groups/' + this.$route.params.id + '/posts' ).then ( response => {
                    if ( ! ( response.data.status == 200 ) ) return;
                    let data = response.data;
                    this.$store.commit ( 'updatePostsData', { key : null, posts : data.data.posts } );
                    setTimeout(() => {
                        let players = document.querySelectorAll('.video-js');
                        for (let [i, player] of players.entries()) {
                            if (!player.hasAttribute("id")) {
                                videojs(player, {
                                    autoplay: false,
                                    controls: true,
                                })
                                break;
                            }
                        }
                        if (this.updateFlag = 1) {
                            this.updateFlag = 0;
                            const updatedPost = document.getElementById(`${postId}`);
                            const videoElem = $(updatedPost).find('div.video-js').attr('id');
                            // console.log(updatedPost);
                            // console.log(data.data.posts);
                            let link;
                            for (let post of data.data.posts) {
                                // console.log(post);
                                if (post.post.id === postId) {
                                    link = post.post.media.attachment_url
                                    console.log(link)
                                    break;
                                }
                            }
                            // console.log(videojs(`#${videoElem}`));
                            // console.log(videojs.getPlayers());
                            videojs(`#${videoElem}`).src({
                                src: link
                            })
                        }
                    }, 300);
                    this.hideLoader ();
                } ).catch ( err => {
                    console.log ( err )
                    this.error ( 'Something went wrong. Please try again later' );
                    this.hideLoader ();
                } );
            },
            removePost ( id ) {
                this.$confirm ( {
                    message : `Are you sure you want to delete this post?`,
                    button : {
                        no : "Cancel",
                        yes : "Yes"
                    },
                    callback : confirm => {
                        if ( confirm ) {
                            this.showLoader ();
                            axios.post ( 'groups/posts/' + id, {
                                _method : "DELETE"
                            } ).then ( ( res ) => {
                                const response = res.data.data;
                                this.getGroupPosts ();
                                this.success ( 'Post Deleted Successfully' );
                            } ).catch ( err => {
                                this.error ( 'Something went wrong. Please try again later' );
                                this.hideLoader ();
                            } )
                        }
                    }
                } );
            },
            editPost ( post ) {
                this.id = post.id;
                this.updateFlag = 1;
                this.postBody = post.body;
                $ ( '[name="ePostBody"]' ).html ( post.body );
                if ( post.media === null ) $ ( "#updatePostForm .post-img-placeholder, #updatePostForm .post-vid-placeholder" ).addClass ( "hide" );
                else {
                    if ( post.extension == "png" || post.extension == "jpeg" || post.extension == "jpg" || post.extension == "jfif" ) {
                        $ ( "#updatePostForm .post-img-placeholder img" ).attr ( "src", post.attachment_url );
                        $ ( "#updatePostForm .post-img-placeholder" ).removeClass ( "hide" );
                        $ ( "#updatePostForm .post-vid-placeholder" ).addClass ( "hide" );
                    } else {
                        $ ( "#updatePostForm .post-img-placeholder" ).addClass ( "hide" );
                        $ ( "#updatePostForm .post-vid-placeholder" ).removeClass ( "hide" );
                        $ ( "#updatePostForm .post-vid-placeholder source" ).attr ( "src", post.attachment_url );
                        $ ( "#updatePostForm .post-vid-placeholder video" ).attr ( "poster", post.media.thumbnail ).removeClass ( "hide" );
                        $ ( "#updatePostForm .post-vid-placeholder video" )[ 0 ].load ();
                    }
                }
            },
            postActionToggle ( event ) {
                let elem = $ ( event.target ).parent ();
                if ( $ ( elem ).siblings ( ".post-actions-list" ).css ( 'visibility' ) == 'hidden' ) {
                    $ ( elem ).siblings ( ".post-actions-list" ).css ( {
                        'visibility' : 'visible', 'opacity' : "1", "transform" : "translateY(0)"
                    } );
                } else {
                    $ ( elem ).siblings ( ".post-actions-list" ).css ( {
                        'visibility' : '', 'opacity' : "", "transform" : ""
                    } );
                }
            },
            toggleComments : function ( e ) {
                $ ( e.target ).parent ().parent ().parent ().siblings ( ".replies-wrapper" ).slideToggle ( 200 );
            },
            replyOnComment : function ( e ) {
                $ ( e.target ).siblings ( ".edit-opinion" ).addClass ( "hide" );
                let parent_id = $ ( e.target ).attr ( "data-parent_id" ),
                    post_id = $ ( e.target ).attr ( "data-post_id" ),
                    elem = $ ( e.target ).parent ().siblings ( ".reply-box-wrapper" ).find ( ".comment-box" );
                this.toggle_comments ( e.target );
                elem.attr ( "data-parent_id", parent_id ).attr ( "data-post_id", post_id );
            },
            toggle_comments ( elem, edit = false ) {
                $ ( elem ).addClass ( "hide" );
                $ ( elem ).siblings ().removeClass ( "hide" );
                if ( $ ( elem ).hasClass ( "reply-btn" ) )
                    $ ( elem ).parent ().siblings ( ".reply-box-wrapper" ).removeClass ( "hide" );

                if ( edit === true ) {
                    let opinion = $ ( elem ).parent ().parent ().siblings ( ".opinion" );
                    opinion.find ( ".opinion-actions a.reply-btn" ).addClass ( "hide" );
                    opinion.find ( ".opinion-actions a.cancel-opinion" ).removeClass ( "hide" );
                    opinion.find ( ".reply-box-wrapper" ).removeClass ( "hide" );
                } else if ( edit === false && ! $ ( elem ).hasClass ( "reply-btn" ) ) {
                    let replyBox = $ ( elem ).parent ().siblings ( ".reply-box-wrapper" );
                    replyBox.addClass ( "hide" );
                    replyBox.find ( ".comment-box" ).html ( "" );
                    replyBox.parent ().siblings ( "opinion-actions .edit-opinion" ).removeClass ( "hide" );
                }
            },
            cancelReply ( event, self = null ) {
                let elem = $ ( event.target );
                if ( elem.length === 0 ) elem = $ ( self );
                this.toggle_comments ( elem );
                $ ( elem ).parent ().siblings ( ".alert" ).remove ();
                let replyBoxWrapper = $ ( this ).parent ().siblings ( ".reply-box-wrapper" );
                replyBoxWrapper.find ( ".post-media-container" ).remove ();
                replyBoxWrapper.find ( ".replyImage" ).val ( "" );
                replyBoxWrapper.find ( ".custom-upload" ).show ();
            },
            onAddingComment ( e, data ) {
                let elem = $ ( e.target );
                if ( e.keyCode === 13 ) {
                    if ( ! ( elem.text ().trim () != "" || elem.prev ( '.post-media-container' ).length > 0 ) ) return;
                    if ( $ ( elem ).parent ().hasClass ( "has-error" ) ) $ ( elem ).siblings ( ".error" ).remove ();
                    let parent_id = typeof ( $ ( elem ).attr ( 'data-parent_id' ) ) === 'undefined' ? null : ( elem ).attr ( 'data-parent_id' );
                    if ( $ ( elem ).hasClass ( "editing" ) === true ) this.axiosComment ( elem, $ ( elem ).attr ( 'data-post-index' ), parent_id, true );
                    else this.axiosComment ( elem, $ ( elem ).attr ( 'data-post-index' ), parent_id, false );
                    elem.blur ();
                    e.preventDefault ();
                }
            },
            prepareCommentData ( elem, parent_id, flag ) {
                let form = new FormData ();
                let replyImage = $ ( elem ).siblings ( ".custom-input-file-container" ).find ( ".replyImage" );
                let comment_image = replyImage.length === 1 ? replyImage[ 0 ].files : 0;
                if ( comment_image.length == 1 ) form.set ( "comment_image", comment_image[ 0 ], comment_image[ 0 ].name );

                // console.log(elem.html ());

                form.set ( "body", elem.html ().replace(/style="[^"]*"/, "") );

                // this.postBody = this.postBody.replace(/style="[^"]*"/, "");

                let mentions = $(elem).find(".atwho-inserted");
                let users = [];
                $.each(mentions, function(i, item) {
                    users.push($(item).attr("data-id"));
                });

                form.set ( "users", users );
                form.set ( "group_id", this.group_data.group.id );
                form.set ( "post_id", typeof elem.attr ( "data-post_id" ) === "undefined" ? null : elem.attr ( "data-post_id" ) );
                
                if ( parent_id == null ) form.set ( "parent_id", null );
                else form.set ( "parent_id", parent_id );
                if ( flag === true ) form.set ( "id", typeof elem.attr ( "data-id" ) === "undefined" ? null : elem.attr ( "data-id" ) );
                if ( flag === true ) form.set ( "_method", "put" );
                return form;
            },
            axiosComment ( elem, index, parent_id, edit ) {
                this.showLoader ();
                const form = this.prepareCommentData ( elem, parent_id, edit );
                let url = "/groups/comments";
                if ( edit === true ) url = url + "/" + elem.attr ( "data-id" );
                axios.post ( url, form ).then ( ( res ) => {
                    this.loadComments ( elem.attr ( "data-post_id" ), index );
                    $ ( elem ).parent ().parent ().parent ().parent ().find ( '.replies-wrapper' ).hide ()
                    let cancel_opinion = $ ( elem ).parent ().parent ().find ( '.opinion-actions' ).find ( '.cancel-opinion' );
                    //sending twice so if first arg1 gets null 2nd must have element
                    this.cancelReply ( cancel_opinion, cancel_opinion );
                    if ( edit === true ) {
                        this.success ( 'Comment Updated Successfully' );
                        elem.removeAttr ( "data-id" ).removeAttr ( "data-post_id" ).removeClass ( "editing" );
                    } else {
                        this.success ( 'Comment Added Successfully' );
                    }
                    elem.text ( "" ).siblings ( ".post-media-container" ).remove ();
                    elem.siblings ( ".custom-input-file-container" ).find ( ".custom-upload" ).show ();
                    $ ( elem ).siblings ( ".custom-input-file-container" ).find ( ".replyImage" ).val ( '' );
                } ).catch ( err => {
                    this.error ( 'Something went wrong. Please try again later' );
                    this.hideLoader ();
                } );
            },
            loadComments ( id, index ) {
                let group_id = this.group_data.group.id;
                const url = "/groups/posts/" + id + "?group_id=" + group_id;
                axios.get ( url ).then ( response => {
                    if ( typeof ( response.data.status ) !== 'undefined' && response.data.status === 200 ) {
                        let data = {
                            comments : response.data.data.comments,
                            totalComments : response.data.data.totalComments
                        };
                        this.$store.commit ( 'updatePostsData', { key : index, response : data } );
                    }
                    this.hideLoader ();
                } ).catch ( err => {
                    console.log ( err )
                    this.error ( 'Something went wrong. Please try again later' );
                    this.hideLoader ();
                } );
            },
            editComment ( e ) {
                let thus = $ ( e.target ).parent ();
                let text = $ ( thus ).attr ( "data-text" ), id = $ ( thus ).attr ( "data-id" ),
                    post_id = $ ( thus ).attr ( "data-post_id" ),
                    elem = $ ( thus ).parent ().parent ().siblings ( ".opinion" ).find ( ".comment-box" );
                this.toggle_comments ( thus, true );
                elem.html ( text );
                elem.attr ( "data-id", id.split ( "comment_id" )[ 1 ] ).attr ( "data-post_id", post_id );
                elem.addClass ( "editing" );
            },
            removeComment ( e ) {
                let id = $ ( e.target ).parent ().attr ( "data-id" );
                id = id.split ( "comment_id" )[ 1 ];
                const url = "groups/comments/" + id;
                this.$confirm ( {
                    message : `Are you sure you want to delete this comment?`,
                    button : {
                        no : "Cancel",
                        yes : "Yes"
                    },
                    callback : confirm => {
                        if ( confirm ) {
                            this.showLoader ();
                            axios.post ( url, {
                                _method : "DELETE"
                            } ).then ( ( res ) => {
                                const response = res.data.data;
                                this.loadComments ( $ ( e.target ).parent ().attr ( "data-post_id" ), $ ( e.target ).parent ().attr ( 'data-post-index' ) );
                                this.success ( 'Comment Deleted Successfully' );
                            } ).catch ( err => {
                                this.error ( 'Something went wrong. Please try again later' );
                                this.hideLoader ();
                            } )
                        }
                    }
                } );
            },
            commentUpload ( e ) {
                let elem = $ ( e.target ).siblings ( 'input[type="file"]' );
                if ( $ ( e.target ).hasClass ( 'icon-photo-camera' ) )
                    elem = $ ( e.target ).parent ().siblings ( 'input[type="file"]' );
                elem.click ();
            },
            replyImage ( e ) {
                let elem = $ ( e.target );
                let input = elem[ 0 ];
                if ( input.files && input.files[ 0 ] ) {
                    let reader = new FileReader ();
                    let imgObj;
                    reader.onload = function ( e ) {
                        const _fileName = $ ( input ).val ().split ( "\\" ).pop ();
                        let commentBoxElem = $ ( input ).parent ().siblings ( ".comment-box" );
                        if ( ! commentBoxElem.text ().trim () ) commentBoxElem.html ( "" );
                        commentBoxElem.focus ();
                        commentBoxElem.trigger ( "propertychange" );
                        const data_lightbox = _fileName.trim ();
                        imgObj = '<div class="post-media-container"><a href="" data-lightbox="" data-title=""><img src=""></a></div>';
                        let inputGparent = $ ( input ).parent ().parent ();
                        inputGparent.prepend ( imgObj );
                        inputGparent.find ( ".post-media-container img" ).attr ( "src", e.target.result );
                        inputGparent.find ( ".post-media-container a" ).attr ( "href", e.target.result );
                        inputGparent.find ( ".post-media-container a" ).attr ( "data-lightbox", data_lightbox );
                        $ ( input ).hide ();
                        $ ( input ).siblings ( ".custom-upload" ).hide ();
                    };
                    reader.readAsDataURL ( input.files[ 0 ] );
                }
            },
            removeCommentMedia ( e ) {
                let key = e.keyCode || e.charCode;
                if ( ( key == 8 || key == 46 ) && $ ( this ).text () == "" ) {
                    $ ( e.target ).siblings ( ".post-media-container" ).remove ();
                    $ ( e.target ).siblings ( ".custom-input-file-container" ).find ( ".custom-upload" ).show ();
                    $ ( e.target ).siblings ( ".custom-input-file-container" ).find ( ".replyImage" ).val ( '' );
                }
            },
            fetch_tagging_users () {
                let id = this.group_data.group.unique_id;
                let url = "/groups/" + id + "/fetch_tagging_users";
                axios.get ( url ).then ( res => {
                    let response = res.data;
                    if ( response.status == 200 && response.message == "success" ) {
                        $ ( document ).on ( "keydown", ".editable", function () {
                            $ ( this ).atwho ( {
                                at : "@",
                                data : response.data,
                                // headerTpl :
                                //     '<div class="atwho-header">Member List<small>↑&nbsp;↓&nbsp;</small></div>',
                                insertTpl : "${name}",
                                displayTpl : "<li><img class='tag-img' src='${profile_pic}'> ${name}</li>",
                                limit : 200
                            } );
                        } );
                    }
                    this.hideLoader ();
                } ).catch ( err => {
                    console.log ( err )
                    this.error ( 'Something went wrong. Please try again later' );
                    this.hideLoader ();
                } );
            },
            viewMemberProfile: function (userId, roleId) {
                this.$emit('viewMemberProfile', userId, roleId);
            },
            replaceNbsp(body) {
                /* body = body.replaceAll('&nbsp;', ' ');
                body = body.replace(/\s\s+/g, ' ');
                body = body.replace(/<div> <\/div>/g, '<br>'); */
                return body
            }
        },
        mounted(){
            $ ( document ).on ( 'paste', '[contenteditable]', function ( e ) {
                e.preventDefault ();
                var text = null;
                // console.log(e.target);

                /* setTimeout(() => {
                    const textarea = $(e.target).hasClass('editable') ? $(e.target) : $(e.target).closest('.editable.form-control');
                    // console.log(textarea);
                    console.log(textarea.find('*'));
                    textarea.find('*').removeAttr('style');
                }, 0); */
                text = ( e.originalEvent || e ).clipboardData.getData ( 'text' ) || prompt ( 'Paste Your Text Here' );
                document.execCommand ( "insertText", false, text );
            } );
            let players = document.querySelectorAll('.video-js');
            for (let [i, player] of players.entries()) {
                videojs(player, {
                    autoplay: false,
                    controls: true,
                })
            }
        }
    }
</script>

<style lang = "scss" scoped>
    @import "resources/assets/sass/abstracts/_variables.scss";

    .post-actions-toggle {
        i.icon-dots {
            font-size: 5px;
        }
    }

    .create-post-wrapper,
    #updatePostForm {
        .profilePic {
            flex: auto;
        }

        .create-post-text.form-control {
            flex: 1 1 100%;
            margin-left: 10px;
        }
    }

    .opinions-wrapper.view-less .opinion-box:nth-of-type(n + 3) {
        display: none;
    }

    .opinions-wrapper:not(.view-less) .view-all-comments {
        display: none;
    }

    .view-actions {
        font-size: 12px;
        padding: 0 20px;
    }

    .view-actions a {
        color: #333;
    }

    .removeMedia .icon-close {
        position: relative;
        top: 2px;
        left: 0;
    }

    .edit-opinion .fa-pencil-square-o:before {
        position: absolute;
        top: 10px;
    }

    .del-opinion .fa-trash:before {
        position: absolute;
        bottom: 10px;
    }

    .post-actions-list {
        position: absolute;
        width: 85px;
        right: -11px;
        top: 30px;
        background: #fff;
        padding: 5px;
        border: 1px solid #e5e5e5;
        box-shadow: 0 3px 5px rgba(0, 0, 0, 0.06);
        opacity: 0;
        visibility: hidden;
        transform: translateY(-25%);
        transition: all ease 0.275s;
        z-index: 1;
    }

    .post-actions-list > a > i {
        padding-right: 5px;
        width: 25px;
    }

    .post-actions-list > a {
        display: flex;
        color: color(dark);
    }

    .post-actions-list:after {
        background: #fff;
        border-bottom: 1px solid #e5e5e5;
        border-left: 1px solid #e5e5e5;
    }

    .cancel-m-dialogue:after, .post-actions-list:after {
        content: "";
        position: absolute;
        bottom: calc(100% - 10px);
        right: 10px;
        height: 20px;
        width: 20px;
        border-radius: 0 0 0 3px;
        transform: rotate(135deg);
    }

    .del-opinion:hover, .del-opinion:focus, .del-post-btn:hover, .del-post-btn:focus {
        color: #c50000;
    }

    .edit-opinion:hover, .edit-opinion:focus {
        color: #009fd6;
    }

    .cancel-opinion {
        font-size: 12px;
    }

    svg:not(:root) {
        overflow: hidden;
    }

    .cancel-m-toggle svg, .thr-dots {
        width: 20px;
        fill: color(secondary);
    }

    .replies-wrapper .reply-wrapper {
        width: calc(100% - 50px);
        margin-left: auto;
        margin-top: 10px;
    }

    .opinion .hide {
        display: none;
    }

    .opinions-wrapper {
        border-bottom: 1px solid color(bg-main);
    }

    .opinion-box {
        background: color(white);
        padding: 15px;
        padding-left: 25px;
        border-radius: 2px;
        margin-bottom: 10px;
    }

    .opinion-header-wrapper {
        display: flex;
        margin-bottom: 10px;
    }

    .opinion {
        flex: 1 1 auto;
        margin: 0 10px;

        p {
            border-radius: 16px;
            overflow-wrap: anywhere;
        }

        .opinion-actions {
            top: 0;
            margin-bottom: 5px;
        }
    }

    .replies-btn {
        color: #5e5e5e;
        display: inline-block;
        position: relative;
        padding-bottom: 4px;
        margin-right: 0px;

        &:after {
            content: "";
            position: absolute;
            background: color(secondary);
            bottom: 0;
            left: 0;
            height: 1px;
            width: 100%;
            transition: width 0.2s ease;
        }
    }

    .reply-btn {
        color: var(--main-color);
        padding: 0;
    }

    .reply-box-wrapper .editable {
        flex: 1 1 auto;
        width: auto;
        padding: 0;
        border: none;
    }

    #comment-box, .comment-box, .opinions-footer .comment-box {
        border: none;
        border-radius: 0;
        background: none;
        height: auto;
    }

    .opinion-actions {
        align-self: flex-start;
        position: relative;
        top: 8px;
    }

    .replies-btn:hover:after {
        width: 20%;
    }

    .opinions-footer {
        position: relative;
        display: flex;
        align-items: flex-start;
        padding: 0 1.5em;
    }

    .opinion-author {
        display: flex;

        a {
            display: flex;
            flex: 0 1 auto;
            color: #333;

            img {
                width: 44px;
                height: 44px;
                min-width: 44px;
                min-height: 44px;
                object-fit: cover;
                border-radius: 50%;
            }
        }
    }

    .opinions-footer .reply-box-wrapper, .reply-box-wrapper {
        flex: 1 1 auto;
        background: #FFFFFF;
        position: relative;
        display: flex;
        align-items: center;
        border: 1px solid #e5e5e5;
        border-radius: 20px;
        padding: 5px 10px;
        flex-wrap: wrap;
        justify-content: flex-end;
        transition: all ease 400ms;
        margin-top: 6px;
    }

    .opinions-footer .reply-box-wrapper {
        margin-left: 10px;
    }

    .reply-box-wrapper:not(.focuses) {
        background: color(bg-main);
    }

    .opinions-footer .reply-box-wrapper .comment-box {
        margin: 0;
        cursor: text;
        flex: 1 1 auto;
        width: auto;
        padding: 0;
        border: none;
        border-radius: 0;
        background: none;
        height: auto;
        overflow-wrap: anywhere;
    }

    [contentEditable="true"]:empty:not(:focus):before {
        content: attr(data-text);
        pointer-events: none;
    }

    .custom-upload {
        cursor: pointer;
    }

    .custom-input-file-container input[type="file"] {
        display: none;
    }

    .imgDots {
        flex: auto;
        text-align: right;
    }

    .postForm .form-group .profilePic img {
        margin-left: 0;
        margin-right: 10px;
    }

    .postForm .hide {
        display: none;
    }

    .wall-col {
        flex: 1 1 60%;
        max-width: 690px;
        padding: 0 16px;
        width: 100%;

        @include mobile {
            padding: 0;
        }
    }

    .wall-side-col {
        flex: 1 1 20%;
    }

    .create-post-text {
        overflow: auto;
        height: auto;
        display: block;
        resize: none;
        border-color: #e5e5e5;
        border-radius: 50px;
        padding: 10px;
        flex: 1 1 auto;
        width: auto;

        span {
            border: none;
            box-shadow: none !important;
            height: auto;
        }
    }

    .post-box {
        padding: 15px;
        background: #fff;
        margin-bottom: 15px;
        box-shadow: $card-shadow;
        border-radius: $border-radius;
    }

    .form-control, select {
        border: 1px solid;
        border-color: #e5e5e5;
        box-shadow: none;
        -webkit-box-shadow: none;
    }

    .author-img {
        height: 50px;
        width: 50px;
        border-radius: 50%;
        object-fit: cover;
    }

    .post-media-container {
        background: #dce4e8;
        border-radius: 2px;
        margin: 7px 0;
        position: relative;

        img {
            max-width: 100%;
            max-height: 400px;
            margin: 0 auto;
            display: block;
        }
    }

    .post-media-container, .post-text {
        flex: 0 0 100%;
    }

    .request-row a.norm-btn,
    .post-img-placeholder a.norm-btn,
    .post-vid-placeholder a.norm-btn,
    .post-iframe-holder a.norm-btn {
        margin: 0 3px;
        font-size: 10px;
        background: #dce3e8;
        padding: 6px 10px;
    }

    .post-vid-placeholder .removeMedia,
    .post-img-placeholder .removeMedia,
    .post-iframe-holder .removeMedia {
        position: absolute;
        right: 3px;
        top: 5px;
    }

    .post-media-container, .post-text {
        flex: 0 0 100%;
    }

    .post-video video, #postMediaVideo, .post-media-container video {
        max-width: 100%;
        max-height: 400px;
        width: 100%;
    }

    video {
        background: #000;
        max-width: 100%;
        display: block;
        margin: 0 auto;
    }

    .post-media-actions-wrapper {
        display: flex;
        justify-content: flex-start;
        margin-top: 8px;
        margin-bottom: 8px;
        padding-left: 40px;
    }

    .custom-upload {
        cursor: pointer;
    }

    .custom-input-file-container input[type="file"] {
        display: none;
    }

    .group-post-media-btn {
        color: #767676;
        display: flex;
        align-items: center;
        font-family: $semiBold;
        cursor: pointer;
    }

    .vee-validate .has-error-span {
        font-size: 12px;
    }

    .commenter-name {
        color: color(dark);
        font-family: $semiBold;
    }

    .comment-body {
        color: color(light);
    }

    .video-js {
        height: auto;
    }
</style>
