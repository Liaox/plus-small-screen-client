<template>
  <div
    v-if="answer"
    class="module-question-list-answer-card"
    @click="gotoAnswerDetail">
    <!-- User avatar. -->
    <user-avatar
      :anonymity="anonymity"
      :src="user.avatar"
      :sex="user.sex" />
    <!-- Body -->
    {{ showUsername }}：{{ body }}
  </div>
  <div v-else class="empty" />
</template>

<script>
import UserAvatar from "./components/UserAvatar.vue";

export default {
  name: "QuestionListAnswerCard",
  components: {
    UserAvatar
  },
  props: {
    answer: {
      type: [Object],
      default: null,
      validator: function(value) {
        if (!value || typeof value === "object") return true;
        return false;
      }
    }
  },
  computed: {
    /**
     * The answer anonymity.
     *
     * @return {Boolean}
     * @author Seven Du <shiweidu@outlook.com>
     */
    anonymity() {
      const { anonymity } = this.answer;
      return !!anonymity;
    },

    /**
     * Get user.
     *
     * @return {Object}
     * @author Seven Du <shiweidu@outlook.com>
     */
    user() {
      const { user } = this.answer;
      return user || {};
    },

    /**
     *  Get show username.
     *
     * @return {string}
     * @author Seven Du <shiweidu@outlook.com>
     */
    showUsername() {
      if (this.anonymity) return "匿名用户";
      return this.user.name;
    },
    body() {
      const { body } = this.answer;
      return body || "";
    }
  },
  methods: {
    gotoAnswerDetail() {
      const { question_id: qid, id } = this.answer;
      this.$router.push(`/questions/${qid}/answers/${id}`);
    }
  }
};
</script>

<style lang="less" scoped>
.module-question-list-answer-card {
  font-size: 30px;
  color: #666;
  overflow: hidden;
  text-overflow: ellipsis;
  word-break: break-all;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  margin-bottom: 30px;
  line-height: 1.4;
}

.empty {
  display: none;
}
</style>
