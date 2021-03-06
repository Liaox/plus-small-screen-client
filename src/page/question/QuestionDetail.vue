<template>
  <div class="p-question-detail">

    <common-header>问题详情</common-header>

    <div class="container">
      <load-more
        ref="questionLoadContainer"
        :on-refresh="handleRefreshAnswers"
        :on-load-more="handleLoadMoreAnswers" >
        <!-- Main -->
        <div class="main">

          <!-- Topics -->
          <div class="main-topics">
            <router-link
              v-for="topic in topics"
              :to="`/question-topics/${topic.id}`"
              :key="topic.id"
              class="label" >
              {{ topic.name }}
            </router-link>
          </div>

          <!-- Title -->
          <h3 class="main-title">{{ question.subject }}</h3>

          <!-- Body -->
          <div class="markdown-body main-body" v-html="htmlBody"/>

          <!-- watch -->
          <div class="main-watch">
            <div class="main-watch-count">
              <span class="follow-count">{{ question.watchers_count }}&nbsp;关注</span>
              <span v-show="question.amount">
                <span>·</span>
                <span class="shang">
                  <span>赏</span>
                  {{ question.amount }}
                </span>
              </span>
            </div>

            <!-- Watch question button -->
            <button
              v-if="question.watched"
              class="main-watch-follow active"
              @click="handleUnwatch" >
              <span>✓</span>已关注
            </button>
            <button
              v-else
              class="main-watch-follow"
              @click="handleWatch" >
              <span>+</span>关注
            </button>
          </div>
        </div>

        <!-- Button -->
        <div class="main-button">
          <div class="button">
            <svg class="main-button-icon" fill="#666">
              <use xlink:href="#base-reward" />
            </svg>
            {{ question.amount ? '已' : '未' }}设置悬赏
          </div>

          <div
            v-if="question.my_answer"
            class="button"
            @click="gotoMyAnswer">
            查看回答
          </div>
          <div
            v-else
            class="button"
            @click="addAnswer">
            <svg class="main-button-icon" fill="#666">
              <use xlink:href="#base-edit" />
            </svg>
            添加回答
          </div>

        </div>

        <!-- Amswers -->
        <div class="answers-tool">
          <div>{{ question.answers_count }}个回答</div>
          <button @click="handleTargetAnswersOrder">
            {{ answersTimeOrder ? '时间排序' : '默认排序' }}
            <svg fill="#999" class="icon">
              <use xlink:href="#base-filter-list" />
            </svg>
          </button>
        </div>

        <!-- Answer list -->
        <question-answer-item
          v-for="answer in answers"
          :key="answer.id"
          :answer="answer" />
      </load-more>
    </div>

    <!-- <div class="tabbar">
      <a class="tabbar-item" href="#">
        <svg class="tabbar-icon" fill="#999">
          <use xlink:href="#message-comments" />
        </svg>
        评论
      </a>
      <a class="tabbar-item" href="#">
        <svg class="tabbar-icon" fill="#999">
          <use xlink:href="#base-share" />
        </svg>
        分享
      </a>
      <a
        v-show="editer"
        class="tabbar-item"
        href="#">
        <svg class="tabbar-icon" fill="#999">
          <use xlink:href="#edit" />
        </svg>
        编辑
      </a>
      <a class="tabbar-item" href="#">
        <svg class="tabbar-icon" fill="#999">
          <use xlink:href="#base-more" />
        </svg>
        更多
      </a>
    </div> -->
  </div>
</template>

<script>
import { render } from "@/util/markdown";
import * as api from "@/api/question/questions";
import { listByDefault, listByTime } from "@/api/question/answer";
import QuestionAnswerItem from "./QuestionAnswerItem";

const noop = () => {};

export default {
  name: "QuestionDetail",
  components: {
    QuestionAnswerItem
  },
  data: () => ({
    question: {},
    answersTimeOrder: false,
    answers: []
  }),
  computed: {
    /**
     * The loged user.
     *
     * @return {Object}
     * @author Seven Du <shiweidu@outlook.com>
     */
    logedUser() {
      const { CURRENTUSER: user } = this.$store.state;
      return user;
    },

    /**
     * The loged user is deiter?
     *
     * @return {boolean}
     * @author Seven Du <shiweidu@outlook.com>
     */
    editer() {
      // 请不要删除，目前暂时不需要管理员可修改问题。
      // 后续需要增加的功能。
      // const { roles = [] } = this.logedUser;
      // for (let index in roles) {
      //   let role = roles[index];
      //   if (role.name === 'founder') {
      //     return true;
      //   }
      // }

      return false;
    },
    topics() {
      const { topics = [] } = this.question;

      return topics;
    },
    htmlBody() {
      const { body = "" } = this.question;

      return render(body);
    },
    loadContainer() {
      return this.$refs.questionLoadContainer;
    },
    answerRequestMethod() {
      if (this.answersTimeOrder) {
        return listByTime;
      }

      return listByDefault;
    }
  },
  watch: {
    answersTimeOrder(newRoute, oldRoute) {
      if (newRoute.path === oldRoute.path) {
        this.answers = [];
        this.loadContainer.beforeRefresh();
      }
    }
  },
  mounted() {
    this.loadContainer.beforeRefresh();
  },
  methods: {
    fetch(cb = noop) {
      api
        .show(this.$route.params.id)
        .then(({ data }) => {
          this.question = data;
          cb();
        })
        .catch(({ response: { data } = {} }) => {
          this.loadContainer.topEnd(false);
          this.loadContainer.bottomEnd(true);
          this.$Message.error(data);
        });
    },
    refreshAnswer() {
      this.answerRequestMethod(this.$route.params.id)
        .then(({ data }) => {
          this.loadContainer.topEnd(false);
          this.loadContainer.bottomEnd(data.length < 15);
          this.answers = data;
        })
        .catch(({ response: { data } = {} }) => {
          this.loadContainer.topEnd(false);
          this.loadContainer.bottomEnd(true);
          this.$Message.error(data);
        });
    },
    handleRefreshAnswers() {
      this.fetch(this.refreshAnswer);
    },
    handleTargetAnswersOrder() {
      this.answersTimeOrder = !this.answersTimeOrder;
    },
    handleWatch() {
      api
        .watch(this.$route.params.id)
        .then(() => {
          this.question.watched = true;
          this.question.watchers_count += 1;
        })
        .catch(({ response: { data } = {} }) => {
          this.$Message.error(data);
        });
    },
    handleUnwatch() {
      api
        .unwatch(this.$route.params.id)
        .then(() => {
          this.question.watched = false;
          this.question.watchers_count -= 1;
        })
        .catch(({ response: { data } = {} }) => {
          this.$Message.error(data);
        });
    },
    handleLoadMoreAnswers() {
      if (!this.answers.length) return;
      this.answerRequestMethod(this.$route.params.id, this.answers.length)
        .then(({ data }) => {
          this.loadContainer.bottomEnd(data.length < 15);
          this.answers = [...this.answers, ...data];
        })
        .catch(({ response: { data } = {} }) => {
          this.loadContainer.bottomEnd(true);
          this.$Message.error(data);
        });
    },
    addAnswer() {
      this.$router.push({ path: `/question/${this.question.id}/answers/add` });
    },
    gotoMyAnswer() {
      const {
        my_answer: { id = "" }
      } = this.question;
      this.$router.push({
        path: `/questions/${this.question.id}/answers/${id}`
      });
    }
  }
};
</script>

<style lang="less" scoped>
.p-question-detail {
  .tabbar {
    position: fixed;
    z-index: 1;
    display: flex;
    justify-content: space-around;
    align-items: center;
    height: 94px;
    width: 100%;
    background-color: #fff;
    bottom: 0;
    border-top: solid 1px #d7d8d8;

    @media screen and (min-width: 769px) {
      width: 768px;
    }

    .tabbar-item {
      flex-grow: 1;
      display: inline-flex;
      justify-content: center;
      align-items: center;
      flex-direction: column;
      color: #999;
      font-size: 12px;
    }

    .tabbar-icon {
      width: 32px;
      height: 32px;
      margin-bottom: 6px;
    }
  }

  .main {
    background-color: #fff;
    padding: 30px;
    width: 100%;
    max-width: 100%;
    height: auto;

    .main-topics {
      width: 100%;
      height: auto;

      > .label {
        display: inline-block;
        background-color: #efefef;
        border-radius: 20px;
        font-size: 24px;
        color: #666;
        padding: 6px 30px;
        margin-right: 14px;
        margin-bottom: 14px;
      }
    }

    .main-title {
      font-size: 36px;
      color: #333;
      font-weight: normal;
      line-height: 1.4;
      margin: 22px 0 26px;
    }

    .main-body {
      font-size: 28px;
      margin-top: 20px;
      color: #666;
    }

    .main-watch {
      width: 100%;
      font-size: 26px;
      color: #999;
      margin-top: 44px;
      display: flex;
      flex-direction: row;
      justify-content: space-between;
      align-items: center;

      .main-watch-follow {
        height: 50px;
        background-color: #fff;
        border-radius: 8px;
        border: solid 2px #59b6d7;
        color: #59b6d7;
        font-size: 26px;
        font-weight: normal;
        font-stretch: normal;
        line-height: 5px;
        letter-spacing: 0px;
        outline: none;

        > span {
          width: 20px;
          height: 20px;
          font-size: 36px;
          margin-right: 2px;
          vertical-align: middle;
        }

        &.active {
          color: #ccc;
          border: solid 2px #ccc;
        }
      }

      .main-watch-count {
        .follow-count {
          margin-right: 12px;
        }

        .shang {
          margin-left: 12px;
          color: #fca308;

          > span {
            color: #fca308;
            width: 20px;
            height: 21px;
            font-size: 22px;
            font-weight: normal;
            font-stretch: normal;
            line-height: 0px;
            letter-spacing: 0px;
            border: solid 1px #fca308;
            padding: 0 4px;
            border-radius: 6px;
          }
        }
      }
    }
  }

  .main-button {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: row;
    border-top: solid 1px #ededed;
    font-size: 28px;
    background-color: #fff;
    padding: 30px 0;

    > .button {
      flex: auto;
      text-align: center;
      display: inline-flex;
      justify-content: center;
      align-items: center;
      color: #666;
      border-right: 1px solid #ededed;

      &:last-child {
        border-right: none;
      }
    }

    .main-button-icon {
      width: 30px;
      height: 30px;
      margin-top: -2px;
      margin-right: 20px;
    }
  }

  .answers-tool {
    position: sticky;
    top: 90+25px;
    background: @gray-bg;
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-size: 24px;
    font-weight: normal;
    font-stretch: normal;
    color: #999;
    padding: 25px 27px;

    > button {
      background-color: transparent;
      outline: none;
      color: #999;

      > .icon {
        width: 30px;
        height: 30px;
        margin-bottom: -6px;
        margin-left: 14px;
      }
    }
  }
}
</style>
