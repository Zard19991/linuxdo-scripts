<template>
  <div class="item">
    <div class="tit">{{ sort }}. 首页新增按创建时间排序</div>
    <input type="checkbox" v-model="localChecked" @change="handleChange" />
  </div>
</template>

<script>
import $ from "jquery";
export default {
  props: {
    value: {
      type: Boolean,
      default: false,
    },
    sort: {
      type: Number,
      required: true,
    },
  },
  data() {
    return {
      localChecked: this.value,
    };
  },
  watch: {
    value(newValue) {
      this.localChecked = newValue;
    },
  },
  methods: {
    handleChange() {
      this.$emit("update:value", this.localChecked);
    },
  },
  created() {
    if (this.localChecked) {
      setInterval(() => {
        if ($(".created_order").length < 1) {
          $("#navigation-bar").prepend(
            `<li title="最新创建" class="latest ember-view created_order"><a href="/latest?order=created">最新创建</a></li>`
          );
          $(".created_order").click(function () {
            setTimeout(() => {
              $("#navigation-bar>li").removeClass("active");
              $(this).addClass("active");
            }, 500);
          });
        }
      }, 1000);
    }
  },
};
</script>
