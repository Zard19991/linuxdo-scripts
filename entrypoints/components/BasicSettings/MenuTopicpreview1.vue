<template>
  <div class="item">
    <div class="tit">{{ sort }}. 是否开启话题预览功能</div>
    <input
      type="checkbox"
      :checked="modelValue"
      @change="$emit('update:modelValue', $event.target.checked)"
    />
  </div>
</template>

<script>
import $ from "jquery";
export default {
  props: ["modelValue", "sort"],
  emits: ["update:modelValue"],
  data() {
    return {
      checked1: false,
      lightboxTimer: null,
      mainTimer: null,
      pollingTimer: null,
    };
  },
  methods: {
    init() {
      if ($(".topicpreview").length < 1) {
        $("body").append(`<div class="topicpreview">
          <div class="topicpreview-opacity"></div>
          <div class="topicpreview-container">
            <p style="text-align: center">正在加载中...</p>  
          </div>
          </div>`);
      }

      $(".topic-list .main-link a.title").each(function () {
        const id = $(this).attr("data-topic-id");
        if ($(this).parents(".link-top-line").find(".topicpreview-btn").length < 1) {
          $(this)
            .parents(".link-top-line")
            .append(
              `<button class="btn btn-icon-text btn-default topicpreview-btn" data-id="${id}">预览</button>`
            );
        }
      });
    },
    setClick() {
      const self = this;
      $(".topicpreview-btn").each(function () {
        $(this).click(function () {
          $(".topicpreview").show();
          let previewData = {};
          let previewurl = $(this).attr("data-id");

          fetch(`/t/${previewurl}.json`)
            .then((response) => response.json())
            .then((data) => {
              previewData = data;

              // 定义一个转化的时间的方法
              function formatDate(isoString) {
                const date = new Date(isoString);
                const year = date.getFullYear();
                const month = String(date.getMonth() + 1).padStart(2, "0"); // 月份从 0 开始，所以要加 1
                const day = String(date.getDate()).padStart(2, "0");
                const hours = String(date.getHours()).padStart(2, "0");
                const minutes = String(date.getMinutes()).padStart(2, "0");
                const seconds = String(date.getSeconds()).padStart(2, "0");
                return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
              }

              $(".topicpreview-container").html(`
                <div class="topicpreview-title">${previewData.title}</div>
                <p class="topicpreview-date">发帖时间：${formatDate(
                  previewData.created_at
                )}</p>
                <div class="topicpreview-content"></div>
                <p style="text-align: center;">仅显示前 20 条，<a class="preview-link" href="/t/topic/${previewurl}/">查看更多</a></p>
              `);

              $.each(previewData.post_stream.posts, function (index, post) {
                $(".topicpreview .topicpreview-content").append(`
                  <div class="item">
                    <span class="itemfloor">${index + 1}楼</span>
                    <div class="itempost">
                      <div class="itemname">
                        ${post.display_username} 
                        <span>${post.username}</span>
                        <div class="itemdate">${formatDate(post.created_at)}</div>
                      </div>
                      ${post.cooked}
                    </div>
                  </div>
                `);
              });

              // 清除之前的定时器
              if (self.lightboxTimer) {
                clearInterval(self.lightboxTimer);
              }
              
              // 创建新定时器并保存引用
              self.lightboxTimer = setInterval(() => {
                $(".lightbox").attr("href", "javascript:void(0)");
              }, 1000);
            });
        });
      });

      // 关闭弹窗
      $(".topicpreview-opacity").click(this.closePreview);
    },
    closePreview() {
      $(".topicpreview").hide();
      $(".topicpreview-container").html(
        `<p style="text-align: center">正在加载中...</p> `
      );
    },
    handleKeyDown(event) {
      if (event.key === "Escape") {
        this.closePreview();
      }
    },
    clearAllTimers() {
      if (this.lightboxTimer) {
        clearInterval(this.lightboxTimer);
        this.lightboxTimer = null;
      }
      if (this.mainTimer) {
        clearInterval(this.mainTimer);
        this.mainTimer = null;
      }
      if (this.pollingTimer) {
        clearInterval(this.pollingTimer);
        this.pollingTimer = null;
      }
    }
  },
  created() {
    if (this.modelValue) {
      this.checked1 = JSON.parse(
        localStorage.getItem("linxudoscriptssettingDMI")
      ).checked1;
      
      this.mainTimer = setInterval(() => {
        if (window.location.href != "https://linux.do/latest?state=muted") {
          this.init();
        }

        $(".preview-link").attr("target", "_blank");
      }, 1000);

      let pollinglength1 = 0;
      let pollinglength2 = 0;
      this.pollingTimer = setInterval(() => {
        if (pollinglength1 != $(".topic-list-body tr").length) {
          pollinglength1 = $(".topic-list-body tr").length;
          this.setClick();
        }
        if (pollinglength2 != $(".post-stream .topic-post").length) {
          pollinglength2 = $(".post-stream .topic-post").length;
          this.setClick();
        }
      }, 1000);
      document.addEventListener("keydown", this.handleKeyDown);
    }
  },
  beforeUnmount() {
    document.removeEventListener("keydown", this.handleKeyDown);
    this.clearAllTimers();
  },
  beforeDestroy() {
    this.clearAllTimers();
  },
};
</script>
