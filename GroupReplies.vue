<template>
    <div class = "reply-wrapper">
        <div class = "opinion-header-wrapper">
            <div class = "opinion-author">
                <a href = "javascript:;">
                    <img :src = "getCommentatorImage(reply.commentator.profile_pic)" />
                </a>
            </div>
            <div class = "opinion">
                <!--Role id is not required so we have to identify all places where we are using viewMemberProfile function-->
                <a class = "commenter-name custom-upload" @click="viewMemberProfile(reply.commentator.id, 'not_required')" href = "javascript:;">{{ reply.commentator.full_name }}&nbsp;</a>
                <p class = "comment-body" v-html = "reply.body"></p>
                <div v-if = "reply.media != null" class = "post-media-container">
                    <a href="javascript:void(0)" :data-title = "reply.body">
                        <img class = "opinion" :src = "reply.attachment_url" @click="replPostImgSrc(reply.attachment_url)" />
                    </a>
                </div>
                <div class = "opinion-actions">
                    <a v-if = "reply.replies.length > 0" href = "javascript:;"
                       class = "replies-btn" v-on:click = "toggleComments">
                        <span>{{ reply.replies.length }}</span>&nbsp;Replies</a>
                    <a href = "javascript:;" v-on:click = "replyOnComment"
                       :data-id = "'comment_id'+reply.id"
                       :data-parent_id = "reply.id"
                       :data-post_id = "post_id" class = "reply-btn">Reply</a>
                    <a href = "javascript:;" v-on:click = "cancelReply($event)"
                       :data-id = "'comment_id'+reply.id"
                       class = "cancel-opinion hide">Cancel</a>
                </div>
                <div class = "reply-box-wrapper hide reply-step-2">
                    <span contenteditable = "true"
                          v-on:keypress = "onAddingComment($event, reply)"
                          v-on:keydown = "removeCommentMedia"
                          @paste = "pasteContent"
                          @cut = "cutEvent"
                          :data-post-index = "postKey" :data-parent_id = "reply.parent_id"
                          class = "editable comment-box form-control"></span>
                    <div class = "custom-input-file-container">
                        <span class = "custom-upload" v-on:click = "commentUpload">
                            <i class = "icon-photo-camera"></i></span>
                        <input type = "file" v-on:change = "replyImage" class = "replyImage" accept = "image/*" />
                    </div>
                </div>
            </div>
            <div class = "opinion-actions">
                <a href = "javascript:;" class = "post-actions-toggle" v-if = "user_id == reply.commentator.id"
                   v-on:click = "postActionToggle">
                    <i class = "icon-dots"></i>
                </a>
                <div class = "post-actions-list">
                    <a v-if = "user_id == reply.commentator.id" v-on:click = "editComment"
                       href = "javascript:;" :data-id = "'comment_id'+reply.id"
                       :data-parent_id = "reply.parent_id" :data-post_id = "post_id"
                       :data-text = "reply.body" class = "edit-opinion">
                        <i class = "fa fa-pencil-square-o"></i>
                        <span>Edit</span>
                    </a>
                    <a v-if = "user_id == reply.commentator.id" href = "javascript:;"
                       v-on:click = "removeComment" :data-post-index = "postKey"
                       :data-id = "'comment_id'+reply.id" :data-post_id = "post_id" class = "del-opinion">
                        <i class = "fa fa-trash"></i>
                        <span>Delete</span>
                    </a>
                </div>
            </div>
        </div>
        <div v-for = "item in reply.replies" class = "replies-wrapper hide" :class = "'comment-box-' + item.id ">
            <GroupReplies :reply = "item" :user_id = "user_id" :post_id = "post_id" :postKey = "postKey"
                          :group_id = "group_id" @viewMemberProfile = "viewMemberProfile" @repImgLink="replPostImgSrc" />
        </div>
    </div>
</template>

<script>

    import axios from "../../store/axios";

    export default {
        name : 'GroupReplies',
        props : [ "reply", "user_id", "post_id", "postKey", "group_id" ],
        data () {
            return {};
        },
        created () {
            $ ( ".post-actions-list" ).each ( function () {
                if ( $ ( this ).css ( 'visibility' ) == 'visible' ) {
                    $ ( this ).css ( {
                        'visibility' : "",
                        'opacity' : "",
                        "transform" : ""
                    } );
                }
            } );
        },
        methods : {
            getCommentatorImage : function ( img ) {
                return ( img ) ? img : '/img/profile-pic.jpeg';
            },
            toggleComments : function ( e ) {
                $ ( e.target ).parent ().parent ().parent ().siblings ( ".replies-wrapper" ).slideToggle ( 200 );
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

                form.set ( "body", elem.html () );
                form.set ( "users", '' );
                form.set ( "group_id", this.group_id );
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
                let group_id = this.group_id;
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
                        console.log ( commentBoxElem )
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
            pasteContent ( e ) {
                let url = e.clipboardData.getData ( "text" ), elem = $ ( e.target );
                if ( $ ( elem ).hasClass ( 'editable' ) == false ) elem = $ ( elem ).closest ( '.editable.form-control' );
                let link = this.$helpers.validateURL ( url );
                if ( ! link ) return;
                if ( $ ( elem ).find ( '.meta-post' ).find ( '.meta-anchor' ).length > 0 ) return;
                // if ( $ ( e.target ).find ( ".meta-anchor" ).length === 1 ) return;
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
            viewMemberProfile: function (userId, roleId) {
                this.$emit('viewMemberProfile', userId, roleId);
            },
            replPostImgSrc: function (replyimgLink) {
                this.$emit('repImgLink', replyimgLink);
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

    .post-actions-list > a > i {
        padding-right: 5px;
        width: 25px;
    }

    .del-opinion:hover, .del-opinion:focus, .del-post-btn:hover, .del-post-btn:focus {
        color: #c50000;
    }

    .edit-opinion:hover, .edit-opinion:focus {
        color: #009fd6;
    }

    .cancel-opinion {
        font-size: 12px;
        color: color(secondary);
        padding: 0 8px;
    }

    svg:not(:root) {
        overflow: hidden;
    }

    .cancel-m-toggle svg, .thr-dots {
        width: 20px;
        fill: #7e7e7e;
    }

    .replies-wrapper .reply-wrapper {
        margin-top: 10px;
    }

    .opinion .hide {
        display: none;
    }

    .opinions-wrapper {
        border-bottom: 1px solid #b7c3c9;
    }

    .opinion-box {
        background: #fff;
        padding: 15px;
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
    }

    .opinion p {
        padding: 8px;
        margin-bottom: 0;
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
        color: color(secondary);
        padding: 0 8px;
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

    .opinion .opinion-actions {
        top: 0;
        margin-bottom: 5px;
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
        min-width: 35px;
    }

    .opinion-author a {
        display: flex;
        flex: 0 1 auto;
        color: #333;
    }

    .opinion-author a img {
        width: 35px;
        height: 35px;
        object-fit: cover;
        border-radius: 50%;
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
    }

    .reply-box-wrapper:not(.focuses) {
        background: #f1f2f5;
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
    }

    [contentEditable="true"]:empty:not(:focus):before {
        content: attr(data-text);
    }

    [contentEditable="true"]:empty:not(:focus):before {
        content: attr(data-text);
    }

    .custom-upload {
        cursor: pointer;
    }

    .custom-input-file-container input[type="file"] {
        display: none;
    }

    .commenter-name {
        color: color(dark);
        font-family: $semiBold;
    }

    .comment-body {
        color: color(light);
    }

</style>
