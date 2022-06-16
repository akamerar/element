<template>
  <transition
    name="dialog-fade"
    @after-enter="afterEnter"
    @after-leave="afterLeave"
  >
    <div
      ref="dialogWrap"
      v-show="visible"
      class="el-dialog__wrapper"
      :class="{ 'is-fullscreen': isFullscreen, 'with-foot': $slots.footer }"
      @click.self="handleWrapperClick"
    >
      <div
        role="dialog"
        :key="key"
        aria-modal="true"
        :aria-label="title || 'dialog'"
        :class="[
          'el-dialog',
          { 'is-fullscreen': fullscreen, 'el-dialog--center': center },
          customClass
        ]"
        ref="dialog"
        :style="style"
        v-resize="onResize"
      >
        <div
          class="el-dialog__header"
          @mousedown="onMouseDown"
          @touchstart="onMouseDown"
        >
          <slot name="title">
            <span class="el-dialog__title">{{ title }}</span>
          </slot>
          <button
            type="button"
            class="el-dialog__headerbtn"
            aria-label="FullScreen"
            v-if="showFullScreen"
            @click="handleFullScreen"
            style="right: 55px"
            :style="isFullscreen ? 'top: 20px' : 'top: 22px'"
          >
            <i
              class="el-dialog__fullscreen"
              :class="
                isFullscreen
                  ? 'iconfont icon-exit-full-screen'
                  : 'el-icon-full-screen'
              "
            ></i>
          </button>
          <button
            type="button"
            class="el-dialog__headerbtn"
            aria-label="Close"
            v-if="showClose"
            @click="handleClose"
          >
            <i class="el-dialog__close el-icon el-icon-close"></i>
          </button>
        </div>
        <div class="el-dialog__body" v-if="rendered"><slot></slot></div>
        <div class="el-dialog__footer" v-if="$slots.footer">
          <slot name="footer"></slot>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import Popup from "element-ui/src/utils/popup";
import Migrating from "element-ui/src/mixins/migrating";
import emitter from "element-ui/src/mixins/emitter";

export default {
  name: "ElDialog",

  mixins: [Popup, emitter, Migrating],

  props: {
    title: {
      type: String,
      default: ""
    },

    modal: {
      type: Boolean,
      default: true
    },

    modalAppendToBody: {
      type: Boolean,
      default: true
    },

    appendToBody: {
      type: Boolean,
      default: false
    },

    lockScroll: {
      type: Boolean,
      default: true
    },

    closeOnClickModal: {
      type: Boolean,
      default: true
    },

    closeOnPressEscape: {
      type: Boolean,
      default: true
    },

    showClose: {
      type: Boolean,
      default: true
    },

    width: String,

    fullscreen: Boolean,

    // 是否可以全屏
    showFullScreen: {
      type: Boolean,
      default: true
    },

    customClass: {
      type: String,
      default: ""
    },

    top: {
      type: String,
      default: "15vh"
    },
    beforeClose: Function,
    center: {
      type: Boolean,
      default: false
    },

    destroyOnClose: Boolean
  },

  data() {
    return {
      isFullscreen: false,
      closed: false,
      key: 0,
      style: {},
      transX: 0,
      transY: 0,
      x: 0,
      y: 0,
      isMobile: false
    };
  },
  watch: {
    visible(val) {
      if (val) {
        this.closed = false;
        this.$nextTick(() => {
          this.getStyle();
        });
        this.$emit("open");
        this.$el.addEventListener("scroll", this.updatePopper);
        this.$nextTick(() => {
          this.$refs.dialog.scrollTop = 0;
        });
        if (this.appendToBody) {
          document.body.appendChild(this.$el);
        }
      } else {
        this.isFullscreen = false;
        this.$el.removeEventListener("scroll", this.updatePopper);
        if (!this.closed) this.$emit("close");
        if (this.destroyOnClose) {
          this.$nextTick(() => {
            this.key++;
          });
        }
      }
    },
    width(val) {
      if (val) {
        this.style.width = this.width;
      }
    }
  },

  computed: {},

  methods: {
    getStyle() {
      let style = {};
      if (!this.fullscreen) {
        if (this.width) {
          style.width = this.width;
        }
      }
      let height = this.$el.querySelector(".el-dialog").offsetHeight;
      let width = this.width;
      if (height == "auto") height = 0;
      if (width.indexOf("%") > 0) width = innerWidth * (parseInt(width) / 100);
      this.transX = parseInt((innerWidth - parseInt(width)) / 2);
      this.transY = parseInt((innerHeight - parseInt(height)) / 2);
      style.left = `${this.transX}px`;
      style.top = `${this.transY}px`;
      this.style = style;
    },
    getMigratingConfig() {
      return {
        props: {
          size: "size is removed."
        }
      };
    },
    handleWrapperClick() {
      if (!this.closeOnClickModal) return;
      this.handleClose();
    },
    handleClose() {
      if (typeof this.beforeClose === "function") {
        this.beforeClose(this.hide);
      } else {
        this.hide();
      }
    },
    handleFullScreen() {
      if (this.isFullscreen) {
        this.isFullscreen = false;
        // exitFullScreen();
      } else {
        this.isFullscreen = true;
        // fullScreen(this.$refs.dialogWrap);
      }
    },
    hide(cancel) {
      if (cancel !== false) {
        this.$emit("update:visible", false);
        this.$emit("close");
        this.isFullscreen = false;
        this.closed = true;
      }
    },
    updatePopper() {
      this.broadcast("ElSelectDropdown", "updatePopper");
      this.broadcast("ElDropdownMenu", "updatePopper");
    },
    afterEnter() {
      this.$emit("opened");
    },
    afterLeave() {
      this.$emit("closed");
    },
    onResize(style, dom) {
      if (style.height === "auto") return;
      this.transX = parseInt((innerWidth - parseInt(style.width)) / 2);
      this.transY = parseInt((innerHeight - parseInt(style.height)) / 2);
      this.style.left = `${this.transX}px`;
      this.style.top = `${this.transY}px`;
      if (this.style.transition !== "top 0.3s ease") {
        this.style.transition = "top 0.3s ease";
        setTimeout(() => {
          this.style.transition = "unset";
        }, 300);
      }
    },
    // 鼠标按下事件
    onMouseDown($event) {
      if (this.isMobile && $event.touches && $event.touches.length) {
        this.x = $event.touches[0].pageX;
        this.y = $event.touches[0].pageY;
        window.addEventListener("touchmove", this.setTrans);
      } else {
        this.x = $event.pageX;
        this.y = $event.pageY;
        window.addEventListener("mousemove", this.setTrans);
      }
    },
    // setTrans
    setTrans(e) {
      var e = e || event;
      let disX = 0;
      let disY = 0;
      if (this.isMobile && e.touches && e.touches.length) {
        disX = e.touches[0].pageX - this.x;
        disY = e.touches[0].pageY - this.y;
        this.x = e.touches[0].pageX;
        this.y = e.touches[0].pageY;
      } else {
        disX = e.pageX - this.x;
        disY = e.pageY - this.y;
        this.x = e.pageX;
        this.y = e.pageY;
      }
      this.transX = this.transX + disX;
      this.transY = this.transY + disY;
      this.style.left = `${this.transX}px`;
      this.style.top = `${this.transY}px`;
    },
    removeEvent() {
      if (this.isMobile) {
        window.removeEventListener("touchmove", this.setTrans);
      } else {
        window.removeEventListener("mousemove", this.setTrans);
      }
    }
  },

  mounted() {
    this.isMobile = "ontouchstart" in window;
    if (this.visible) {
      this.rendered = true;
      this.open();
      if (this.appendToBody) {
        document.body.appendChild(this.$el);
      }
    }
    if (this.isMobile) {
      window.addEventListener("touchend", this.removeEvent);
    } else {
      window.addEventListener("mouseup", this.removeEvent);
    }
  },

  destroyed() {
    // if appendToBody is true, remove DOM node after destroy
    if (this.appendToBody && this.$el && this.$el.parentNode) {
      this.$el.parentNode.removeChild(this.$el);
    }
    if (this.isMobile) {
      window.removeEventListener("touchend", this.removeEvent);
    } else {
      window.removeEventListener("mouseup", this.removeEvent);
    }
  }
};
</script>
