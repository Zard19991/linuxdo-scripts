<template>
  <div class="item">
    <div class="tit">{{ sort }}. 是否自动展开回复父帖子</div>
    <input type="checkbox" :checked="modelValue" @change="$emit('update:modelValue', $event.target.checked)" />
  </div>
</template>

<script>
import $ from "jquery";
export default {
  props: ["modelValue", "sort"],
  emits: ["update:modelValue"],
  data() {
    return {
      replyParentIntervalId: null // 添加变量存储定时器ID
    };
  },
  methods: {
    init() {
      const tabs = $(".reply-to-tab");
      let index = 0;

      const clickNext = () => {
        if (index >= tabs.length) return;

        const tab = $(tabs[index]);
        if (!tab.data("clicked")) {
          tab.click();
          tab.data("clicked", true);
        }
        index++;
        setTimeout(clickNext, 1000);
      };
      clickNext();
    },
  },
  created() {
    if (this.modelValue) {
      this.replyParentIntervalId = setInterval(() => {
        if ($(".topic-body .reply-to-tab").length > 0) {
          this.init();
        }
      }, 1000);
    }
  },
  beforeUnmount() {
    // 清除定时器
    if (this.replyParentIntervalId) {
      clearInterval(this.replyParentIntervalId);
    }
  },
  // Vue 2 兼容性
  beforeDestroy() {
    // 清除定时器
    if (this.replyParentIntervalId) {
      clearInterval(this.replyParentIntervalId);
    }
  }
};
</script>
