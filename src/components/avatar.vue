<template>
  <router-link
    :to="path"
    :class="styles"
    class="m-flex-shrink0 m-flex-grow0 m-avatar-box"
    @click.native.stop>
    <img
      v-if="avatar"
      :src="avatar"
      class="m-avatar-img"
      @error="handelError">
    <i
      v-if="icon"
      :style="icon"
      class="m-avatar-icon"/>
  </router-link>
</template>

<script>
import _ from "lodash";

export default {
  name: "Avatar",
  props: {
    size: { type: String, default: "def" },
    user: { type: Object, required: true }
  },
  computed: {
    uid() {
      return this.user.id;
    },
    sex() {
      return ~~this.user.sex;
    },
    icon() {
      const { verified = {} } = this.user;
      if (_.isEmpty(verified)) return false;
      // 如果有设置图标 使用设置的图标
      if (verified.icon)
        return { "background-image": `url("${verified.icon}")` };
      // 否则根据认证类型使用相应的默认图标
      else if (verified.type === "user")
        return {
          "background-image": "url(" + require("@/images/cert_user.png") + ")"
        };
      else if (verified.type === "org")
        return {
          "background-image": "url(" + require("@/images/cert_org.png") + ")"
        };
      // 如果没有认证 就不显示
      else return false;
    },
    path() {
      return this.uid ? `/users/${this.uid}` : "javascript:;";
    },
    styles() {
      const sex = ["secret", "man", "woman"];
      return this.avatar
        ? [`m-avatar-box-${this.size}`]
        : [`m-avatar-box-${this.size}`, `m-avatar-box-${sex[this.sex]}`];
    },
    avatar: {
      get() {
        return this.user.avatar;
      },
      set(val) {
        this.user.avatar = val;
      }
    }
  },
  methods: {
    handelError() {
      this.avatar = null;
    }
  }
};
</script>
