
<template>
  <div :class="$style.wrapper" id="scroller" style="height: 100%">
    <div v-show="!isAtBeginning" :class="[$style.control, $style.prev]" :style="controlOffsetStyle" @click="prev">
      <Arrow />
    </div>
    <div :class="$style.container" ref="container" style="height:100%">
      <div ref="scroller" :style="{ width: `${scrollerWidth}px`, height: `${scrollerHeight}px` }" id="scroller">
        <div
          ref="visibleItems"
          :id="'visibleItems'+index"
          v-for="(item, index) in visibleItems"
          :key="index"
          :class="$style.visibleItems"
          :style="{ transform: `translate3d(${item.virtualProps && item.virtualProps.left || 0}px, 0, 0)`, width: item.virtualProps && item.virtualProps.width }"
        >
          <slot name="item" :item="item">{{ item }}</slot>
        </div>
      </div>
    </div>
    <div v-show="!isAtEnd" :class="[$style.control, $style.next]" :style="controlOffsetStyle" @click="next">
      <Arrow />
    </div>
    <!-- Render one hidden item element off the screen so we can measure height -->
    <div
        id="measureit"
        ref="itemToMeasure"
        :key="index"
        v-for="(item, index) in items.slice(0, 1)"
        :style="{ width: itemTotalWidth - itemSpacing + 'px', opacity: '0', top: '-1000px', position: 'relative' }"
      >
        <slot name="item" :item="item">{{ item }}</slot>
    </div>
  </div>
</template>

<script>
import throttle from 'lodash.throttle';
import smoothscroll from 'smoothscroll-polyfill';
import Arrow from './Arrow.vue';

export default {
  components: {
    Arrow,
  },

  data() {
    return {
      ready: false,
      controlOffset: 0,
      itemTotalWidth: 0,
      scrollLeft: 0,
      scrollerWidth: 0,
      scrollerHeight: 0,
      containerWidth: 0,
    };
  },

  props: {
    items: {
      // List of items to be rendered
      type: Array,
      required: true,
    },
    controlTargetClass: {
      type: String,
      default: 'js-scroller-control-target',
    },
    breakpoints: {
      type: Array,
      default() {
        return [
          {
            documentWidth: 0,
            itemMinWidth: 150,
          },
          {
            documentWidth: 1024,
            itemMinWidth: 250,
          },
        ];
      },
    },
    itemSpacing: {
      type: Number,
      default: 15,
    },
  },


  watch: {
    items(arr) {
      if (arr.length) {
        this.resetScroller();
      }
    },
  },

  methods: {
    bindScroller() {
      this.throttledResetScroller = throttle(() => this.resetScroller(), 100);

      window.addEventListener('resize', this.throttledResetScroller);
      this.$refs.container.addEventListener('scroll', this.updateScrollData);
    },

    resetScroller() {
      this.containerWidth = this.$refs.container.offsetWidth;

      let itemMinWidth = 0;

      this.breakpoints.forEach((breakpoint) => {
        if (document.body.clientWidth > breakpoint.documentWidth) {
          itemMinWidth = breakpoint.itemMinWidth;
        }
      });

      const itemMinTotalWidth = itemMinWidth + this.itemSpacing;
      const numItems = Math.floor((this.containerWidth + this.itemSpacing) / itemMinTotalWidth);
      const itemWidth = this.containerWidth / numItems - this.itemSpacing + this.itemSpacing / numItems;

      this.scrollerWidth = Math.floor((itemWidth + this.itemSpacing) * this.items.length - this.itemSpacing);

      let itemHeight = 0;

      // loop through each item and update left position and width
      this.items.forEach((item, index) => {
        item.virtualProps = {
         width: `${itemWidth}px`,
         left: (itemWidth + this.itemSpacing) * index,
        }
      });

      this.itemTotalWidth = itemWidth + this.itemSpacing;

      this.$nextTick().then(() => {
        this.scrollerHeight = this.$refs.itemToMeasure[0].offsetHeight;
      });


      this.setControlOffset();
    },

    setControlOffset() {

      this.$nextTick().then(() => {
        //this math wont work if target element isnt first, if image is at bottom and text is at top calcs are off, need to fix it later
        const controlSize = 40;
        const controlTargetEl = this.$refs.itemToMeasure[0].querySelector(`.${this.controlTargetClass}`) || this.$refs.itemToMeasure[0];
        const targetHeight = controlTargetEl.offsetHeight;

        this.controlOffset = targetHeight / 2 - controlSize / 2;
        console.log(this.$refs.itemToMeasure[0].querySelector(`.${this.controlTargetClass}`));
        console.log('this.controlOffset', this.controlOffset);
      });
    },

    updateScrollData() {
      this.scrollLeft = this.$refs.container.scrollLeft;
    },

    prev() {
      const left = Math.max(
        this.scrollLeft - (this.scrollLeft % this.itemTotalWidth) - this.containerWidth - this.itemSpacing,
        0,
      );

      this.smoothScrollTo(left);
    },

    next() {
      const progress = this.scrollLeft + this.containerWidth;

      const left = this.scrollLeft + this.containerWidth + this.itemTotalWidth - (progress % this.itemTotalWidth);

      this.smoothScrollTo(left);
    },

    smoothScrollTo(left) {
      this.$refs.container.scroll({ top: 0, left, behavior: 'smooth' });
    },
  },

  computed: {
    controlOffsetStyle() {
      if (this.controlOffset) {
        return {
          top: `${this.controlOffset}px`,
        };
      }
    },

    visibleItems() {
      const startPosition = Math.floor(this.scrollLeft / this.itemTotalWidth)
      const endPosition = Math.floor( ( this.scrollLeft + this.containerWidth ) / this.itemTotalWidth );
      console.log('startPosition', startPosition);
      console.log('endPosition', endPosition);

      const sliceStart = Math.max(0, startPosition - 2);
      const sliceEnd = Math.min(this.items.length, endPosition + 2);
      console.log('sliceStart', sliceStart);
      console.log('sliceEnd', sliceEnd);
      return this.items.slice(sliceStart, sliceEnd);
    },

    isAtBeginning() {
      return this.scrollLeft === 0;
    },

    isAtEnd() {
      return this.scrollLeft >= this.scrollerWidth - this.containerWidth;
    },
  },

  mounted() {
    //this.items = this.$children.filter((component) => component.$options._componentTag === 'ScrollerItem');
    smoothscroll.polyfill();

    this.resetScroller();
    this.bindScroller();
  },

  beforeDestroy() {
    window.removeEventListener('resize', this.throttledResetScroller);
    this.$refs.container.removeEventListener('scroll', this.updateScrollData);
  },
};
</script>

<style module lang="scss">

$control-size: 40px;

.wrapper {
  position: relative;
}

.container {
  -ms-overflow-style: none;
  overflow-x: scroll;
  overflow-y: hidden;
  position: relative;
  /* stylelint-disable */
  scrollbar-width: none;
  /* stylelint-enable */

  &::-webkit-scrollbar {
    height: 0;
    width: 0;
  }
}

.items > * {
  display: inline-block;
}

.visibleItems {
  display: inline-block;
  position: absolute;
  top: 0;
  left: 0;
}

.control {
  background: white;
  border-radius: 100px;
  box-shadow: 0 3px 6px rgba(black, 0.16);
  cursor: pointer;
  height: $control-size;
  position: absolute;
  width: $control-size;
  z-index: 100;

  :global(svg) {
    fill: gray;
    height: 16px;
    left: 16px;
    position: absolute;
    top: 12px;
    width: 10px;
  }

  &.prev {
    left: -$control-size / 2;

    :global(svg) {
      left: auto;
      right: 16px;
      transform: scaleX(-1);
    }
  }

  &.next {
    right: -$control-size / 2;
  }
}
</style>
