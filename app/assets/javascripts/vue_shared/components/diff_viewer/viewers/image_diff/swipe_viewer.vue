<script>
import _ from 'underscore';
import { pixeliseValue } from '../../../lib/utils/dom_utils';
import ImageViewer from '../../../content_viewer/viewers/image_viewer.vue';

export default {
  components: {
    ImageViewer,
  },
  props: {
    newPath: {
      type: String,
      required: true,
    },
    oldPath: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      dragging: false,
      swipeOldImgInfo: null,
      swipeNewImgInfo: null,
      swipeMaxWidth: undefined,
      swipeMaxHeight: undefined,
      swipeBarPos: 1,
      swipeWrapWidth: 0,
    };
  },
  computed: {
    swipeMaxPixelWidth() {
      return pixeliseValue(this.swipeMaxWidth);
    },
    swipeMaxPixelHeight() {
      return pixeliseValue(this.swipeMaxHeight);
    },
    swipeWrapPixelWidth() {
      return pixeliseValue(this.swipeWrapWidth);
    },
    swipeBarPixelPos() {
      return pixeliseValue(this.swipeBarPos);
    },
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.resizeThrottled, false);
    document.body.removeEventListener('mouseup', this.stopDrag);
    document.body.removeEventListener('mousemove', this.dragMove);
  },
  mounted() {
    window.addEventListener('resize', this.resize, false);
  },
  methods: {
    dragMove(e) {
      if (!this.dragging) return;

      let leftValue = e.pageX - this.$refs.swipeFrame.getBoundingClientRect().left;
      const spaceLeft = 20;
      const { clientWidth } = this.$refs.swipeFrame;
      if (leftValue <= 0) {
        leftValue = 0;
      } else if (leftValue > clientWidth - spaceLeft) {
        leftValue = clientWidth - spaceLeft;
      }

      this.swipeWrapWidth = (leftValue / clientWidth) * 100;
      this.swipeBarPos = leftValue;
    },
    startDrag() {
      this.dragging = true;
      document.body.addEventListener('mousemove', this.dragMove);
    },
    stopDrag() {
      this.dragging = false;
      document.body.removeEventListener('mousemove', this.dragMove);
    },
    prepareSwipe() {
      if (this.swipeOldImgInfo && this.swipeNewImgInfo) {
        // Add 2 for border width
        this.swipeMaxWidth =
          Math.max(this.swipeOldImgInfo.renderedWidth, this.swipeNewImgInfo.renderedWidth) + 2;
        this.swipeMaxHeight =
          Math.max(this.swipeOldImgInfo.renderedHeight, this.swipeNewImgInfo.renderedHeight) + 2;

        document.body.addEventListener('mouseup', this.stopDrag);
      }
    },
    swipeNewImgLoaded(imgInfo) {
      this.swipeNewImgInfo = imgInfo;
      this.prepareSwipe();
    },
    swipeOldImgLoaded(imgInfo) {
      this.swipeOldImgInfo = imgInfo;
      this.prepareSwipe();
    },
    resize: _.throttle(function throttledResize() {
      this.swipeBarPos = 0;
    }, 400),
  },
};
</script>

<template>
  <div class="swipe view">
    <div
      ref="swipeFrame"
      :style="{
        'user-select': dragging ? 'none' : null,
      }"
      class="swipe-frame"
    >
      <image-viewer
        key="swipeOldImg"
        ref="swipeOldImg"
        :render-info="false"
        :path="oldPath"
        class="frame deleted"
        @imgLoaded="swipeOldImgLoaded"
      />
      <div
        ref="swipeWrap"
        :style="{
          width: `${swipeWrapWidth}%`,
        }"
        class="swipe-wrap"
      >
        <image-viewer
          key="swipeNewImg"
          :render-info="false"
          :path="newPath"
          :style="{
            width: swipeMaxPixelWidth,
          }"
          class="frame added"
          @imgLoaded="swipeNewImgLoaded"
        >
          <slot slot="image-overlay" name="image-overlay"> </slot>
        </image-viewer>
      </div>
      <span
        ref="swipeBar"
        :style="{ left: swipeBarPixelPos }"
        class="swipe-bar"
        @mousedown="startDrag"
        @mouseup="stopDrag"
      >
        <span class="top-handle"></span> <span class="bottom-handle"></span>
      </span>
    </div>
  </div>
</template>
