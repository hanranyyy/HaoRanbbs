<template>
    <div class="comment-item">
      <Avatar :userId="commentData.userId" :width="50"></Avatar>
      <div class="comment-info">
        <div class="nick-name">
          <span class="name" @click="gotoUcenter(commentData.userId)">{{
            commentData.nickName
          }}</span>
          <span v-if="commentData.userId == articleUserId" class="tag-author"
            >作者</span
          >
        </div>
        <div class="comment-content">
          <div>
            <span class="tag tag-topping" v-if="commentData.topType == 1"
              >置顶</span
            >
            <span class="tag no-audit" v-if="commentData.status==0">待审核</span>
            <span v-html="commentData.content"></span>
          </div>
          <CommentImage
            :style="{ marginTop: '10px' }"
            v-if="commentData.imgPath"
            :src="
              proxy.globalInfo.imageUrl + commentData.imgPath.replace('.', '_.')
            "
            :imgList="[proxy.globalInfo.imageUrl + commentData.imgPath]"
          />
        </div>
        <div class="op-panel">
          <div class="time">
            <span>{{ commentData.postTime }}</span>
            <span>&nbsp;&nbsp;·&nbsp;&nbsp;{{ commentData.userIpAddress }} </span>
          </div>
          <div
            :class="[
              'iconfont icon-good',
              commentData.likeType == 1 ? 'active' : '',
            ]"
            @click="doLike(commentData)"
          >
            {{ commentData.goodCount > 0 ? commentData.goodCount : "点赞" }}
          </div>
          <div
            class="iconfont icon-comment"
            @click="showReplyPanel(commentData, 0)"
          >
            回复
          </div>
          <el-dropdown v-if="articleUserId == currentUserId">
            <div class="iconfont icon-more"></div>
            <template #dropdown>
              <el-dropdown-menu>
                <el-dropdown-item @click="opTop(commentData)">
                  {{ commentData.topType == 0 ? "设为置顶" : "取消置顶" }}
                </el-dropdown-item>
              </el-dropdown-menu> 
            </template>
          </el-dropdown>
        </div>
        <div class="comment-sub-list" v-if="commentData.children">
          <div class="comment-sub-item" v-for="sub in commentData.children">
            <Avatar :userId="sub.userId" :width="30"></Avatar>
            <div class="comment-sub-info">
              <div class="nick-name">
                <span class="name" @click="gotoUcenter(sub.userId)">{{
                  sub.nickName
                }}</span>
                <span
                  v-if="commentData.userId == articleUserId"
                  class="tag-author"
                  >作者</span
                >
                <span class="reply-name">回复：</span>
                <span @click="gotoUcenter(sub.replyUserId)" class="a-link"
                  >@{{ sub.replyNickName }}：</span
                >
                <span class="sub-content" v-html="sub.content"></span>
              </div>
              <div class="op-panel">
                <div class="time">
                  <span>{{ commentData.postTime }}</span>
                  <span
                    >&nbsp;&nbsp;·&nbsp;&nbsp;{{ commentData.userIpAddress }}
                  </span>
                </div>
                <div
                  :class="[
                    'iconfont icon-good',
                    sub.likeType == 1 ? 'active' : '',
                  ]"
                  @click="doLike(sub)"
                >
                  {{ sub.goodCount > 0 ? sub.goodCount : "点赞" }}
                </div>
                <div
                  class="iconfont icon-comment"
                  @click="showReplyPanel(sub, 1)"
                >
                  回复
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="reply-info" v-if="commentData.showReply">
          <CommentPost
            :articleId="articleId"
            :avatarWidth="30"
            :userId="currentUserId"
            :showInsertImg="false"
            :pCommentId="pCommentId"
            :replyUserId="replyUserId"
            @postCommentFinish="postCommentFinish"
            :placeholderInfo="placeholderInfo"
          />
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { defineEmits, ref, getCurrentInstance } from "vue";
  import CommentPost from "./CommentPost.vue";
  import CommentImage from "./CommentImage.vue";
  import { useRouter } from "vue-router";
  const { proxy } = getCurrentInstance();
  const router = useRouter();
  
  const props = defineProps({
    articleId: {
      type: String,
    },
    commentData: {
      type: Object,
    },
    articleUserId: {
      type: String,
    },
    currentUserId: {
      type: String,
    },
  });
  const api = {
    doLike: "/comment/doLike",
    changeTopType: "/comment/changeTopType",
  };
  const emits = defineEmits(["hiddenAllReply", "reloadData"]);
  //显示回复框
  const showReplyPanel = (curData, type) => {
    const haveShow =
      props.commentData.showReply == undefined
        ? false
        : props.commentData.showReply;
    emits("hiddenAllReply");
    if (type == 0) {
      props.commentData.showReply = !haveShow;
    } else {
      props.commentData.showReply = true;
    }
    pCommentId.value = props.commentData.commentId;
    replyUserId.value = curData.userId;
    placeholderInfo.value = "回复 @" + curData.nickName;
  };
  //回复的ID
  const pCommentId = ref(0);
  const replyUserId = ref(null);
  
  const postCommentFinish = (resultData) => {
    props.commentData.children = resultData;
    placeholderInfo.value = undefined;
  };
  
  const placeholderInfo = ref(null);
  
  const gotoUcenter = (userId) => {
    router.push(`/user/${userId}`);
  };
  //点赞
  const doLike = async (data) => {
    let result = await proxy.Request({
      url: api.doLike,
      showLoading: false,
      params: {
        commentId: data.commentId,
      },
    });
    if (!result) {
      return;
    }
    data.goodCount = result.data.goodCount;
    data.likeType = result.data.likeType;
  };
  // 置顶
  const opTop = async (data) => {
    let result = await proxy.Request({
      url: api.changeTopType,
      params: {
        commentId: data.commentId,
        topType: data.topType == 1 ? 0 : 1,
      },
    });
    if (!result) {
      return;
    }
    emits("reloadData");
  };
  </script>
  
  <style lang="scss" scoped>
  .comment-item {
    display: flex;
    padding-top: 15px;
    .comment-info {
      margin-left: 10px;
      border-bottom: 1px solid #ddd;
      padding-bottom: 15px;
      flex: 1;
      .nick-name {
        .name {
          font-size: 14px;
          color: var(--text2);
          margin-right: 10px;
          cursor: pointer;
        }
        .tag-author {
          background: var(--pink);
          color: #fff;
          font-size: 12px;
          border-radius: 2px;
          padding: 0px 3px;
        }
      }
      .comment-content {
        margin-top: 5px;
        font-size: 15px;
        line-height: 22px;
        .tag {
          font-size: 12px;
          border-radius: 3px;
          padding: 0px 5px;
          margin-right: 5px;
        }
        .tag-topping {
          color: var(--pink);
          border: 1px solid var(--pink);
        }
        .no-audit {
          color: var(--text2);
          border: 1px solid var(--text2);
        }
      }
      .op-panel {
        display: flex;
        margin-top: 5px;
        font-size: 13px;
        color: var(--text2);
        align-items: center;
        .time {
          margin-right: 20px;
        }
        .iconfont {
          margin-right: 15px;
          font-size: 14px;
          color: var(--icon);
          cursor: pointer;
        }
        .iconfont::before {
          margin-right: 3px;
        }
        .active {
          color: var(--link);
        }
      }
      .comment-sub-list {
        margin-top: 10px;
        .comment-sub-item {
          display: flex;
          margin: 8px 0px;
          font-size: 14px;
          .comment-sub-info {
            margin-left: 5px;
            .nick-name {
              .reply-name {
                margin: 0px 5px;
              }
              .sub-content {
                font-size: 15px;
              }
            }
          }
        }
      }
      .reply-info {
        margin-top: 15px;
      }
    }
  }
  </style>
  