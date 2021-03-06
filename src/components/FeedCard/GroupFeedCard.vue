<script>
import bus from "@/bus.js";
import FeedCard from "./FeedCard.vue";
import * as api from "@/api/group.js";

export default {
  name: "GroupFeedCard",
  extends: FeedCard,
  computed: {
    liked: {
      get() {
        return !!this.feed.liked;
      },
      set(val) {
        this.feed.liked = val;
      }
    },
    commentCount: {
      get() {
        return this.feed.comments_count || 0;
      },
      set(val) {
        this.feed.comments_count = ~~val;
      }
    },
    likeCount: {
      get() {
        return this.feed.likes_count || 0;
      },
      set(val) {
        this.feed.likes_count = ~~val;
      }
    },
    body() {
      return this.feed.summary || "";
    },
    images() {
      let images = [];
      this.feed.images.map(image => {
        image.file = image.id;
        images = [...images, image];
      });

      return images || [];
    },
    has_collect: {
      get() {
        return this.feed.collected || 0;
      },
      set(val) {
        this.feed.collected = val;
      }
    },
    viewCount() {
      return this.feed.views_count || 0;
    }
  },
  methods: {
    handleView() {
      this.$router.push(`/groups/${this.feed.group.id}/posts/${this.feed.id}`);
    },
    handleLike() {
      const method = this.liked ? "delete" : "post";
      const url = `/plus-group/group-posts/${this.feed.id}/likes`;
      if (this.fetching) return;
      this.fetching = true;
      this.$http({
        method,
        url,
        validateStatus: s => s === 201 || s === 204
      })
        .then(() => {
          method === "post"
            ? ((this.liked = true), (this.likeCount += 1))
            : ((this.liked = false), (this.likeCount -= 1));
          this.fetching = false;
        })
        .catch(() => {
          this.fetching = false;
        });
    },
    sendComment({ reply_user: replyUser, body }) {
      if (body && body.length === 0)
        return this.$Message.error("评论内容不能为空");

      const params = {
        body,
        reply_user: replyUser
      };
      api.postComment(this.feed.id, params).then(comment => {
        this.commentCount += 1;
        this.comments.unshift(comment);
        this.$Message.success("评论成功");
        bus.$emit("commentInput:close", true);
      });
    },
    handleCollection() {
      api.collectGroupPost(this.feed.id, this.has_collect).then(() => {
        this.$Message.success("操作成功");
        this.has_collect = !this.has_collect;
      });
    },
    handleMore() {
      const actions = [];
      if (this.has_collect) {
        actions.push({
          text: "取消收藏",
          method: () => {
            api.uncollectPost(this.feed.id).then(() => {
              this.$Message.success("取消收藏");
              this.has_collect = false;
            });
          }
        });
      } else {
        actions.push({
          text: "收藏",
          method: () => {
            api.collectionPost(this.feed.id).then(() => {
              this.$Message.success("已加入我的收藏");
              this.has_collect = true;
            });
          }
        });
      }
      if (this.isMine) {
        // 是否是自己文章
        actions.push({
          text: "申请文章置顶",
          method: () => {
            bus.$emit("applyTop", {
              type: "post",
              api: api.applyTopPost,
              payload: this.feed.id
            });
          }
        });
        actions.push({
          text: "删除帖子",
          method: () => {
            setTimeout(() => {
              const actionSheet = [
                {
                  text: "删除",
                  style: { color: "#f4504d" },
                  method: () => {
                    api
                      .deletePost(this.feed.group.id, this.feed.id)
                      .then(() => {
                        this.$Message.success("删除帖子成功");
                        this.$nextTick(() => {
                          this.$el.remove();
                          this.$emit("afterDelete");
                        });
                      });
                  }
                }
              ];
              bus.$emit("actionSheet", actionSheet, "取消", "确认删除?");
            }, 200);
          }
        });
      } else {
        actions.push({
          text: "举报",
          method: () => {
            this.$Message.info("举报功能开发中，敬请期待");
          }
        });
      }

      bus.$emit("actionSheet", actions, "取消");
    },
    commentAction({ isMine = false, placeholder, reply_user, comment }) {
      if (isMine) {
        const isOwner = this.feed.user.id === this.CURRENTUSER.id;
        bus.$emit("actionSheet", [
          {
            text: isOwner ? "评论置顶" : "申请评论置顶",
            method: () => {
              bus.$emit("applyTop", {
                isOwner,
                type: "postComment",
                api: api.applyTopPostComment,
                payload: { postId: this.feedID, commentId: comment.id }
              });
            }
          },
          { text: "删除评论", method: () => this.deleteComment(comment.id) }
        ]);
      } else {
        this.handleComment({
          placeholder,
          reply_user
        });
      }
    },
    deleteComment(commentId) {
      api.deletePostComment(this.feedID, commentId).then(() => {
        this.feed.comments = this.feed.comments.filter(c => c.id !== commentId);
        this.commentCount -= 1;
        this.$Message.success("删除评论成功");
      });
    }
  }
};
</script>
