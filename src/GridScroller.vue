<!--
  ADOBE CONFIDENTIAL
  ___________________

  Copyright 2019 Adobe
  All Rights Reserved.

  NOTICE: All information contained herein is, and remains
  the property of Adobe and its suppliers, if any. The intellectual
  and technical concepts contained herein are proprietary to Adobe
  and its suppliers and are protected by all applicable intellectual
  property laws, including trade secret and copyright laws.
  Dissemination of this information or reproduction of this material
  is strictly forbidden unless prior written permission is obtained
  from Adobe.
-->

<template>
  <div>
    <div v-if="renderedItems.length" ref="scroller"  :style="{ 'padding-top': '500px' }">
      <div
        v-for="(item, index) in renderedItems"
        :id="`scroller-item-${item.itemIndex}`"
        :key="`scroller-item-${index}`"
        :class="{ [$style.item]: mounted }"
        :style="getItemStyle(item)"
        ref="items"
      >
        <slot name="item" :item="item" :index="index+1" />
      </div>
    </div>
  </div>
</template>

<script>
import throttle from 'lodash.throttle';

export default {
  data() {
    return {
      mounted: false,
      itemSpacing: 20,
      virtualizerBufferSize: 2,
      controlOffset: 0,
      itemTotalWidth: 0,
      itemWidth: 0,
      itemHeight: 0,
      numItems: 0,
      scrollLeft: 0,
      scrollerWidth: 0,
      scrollerHeight: 0,
      containerWidth: 0,
      measuredItems: [],
      numSsrCards: 20,
      paddingTop: 0,
      scrollTop: 0,
      breakpoints: [
        {
          documentWidth: 0,
          itemMinWidth: 150,
        },
        {
          documentWidth: 1024,
          itemMinWidth: 250,
        },
      ],
      currentBreakpoint: {},
    };
  },

  props: {
    items: {
      type: Array,
      required: true,
    },
    controlTargetClass: {
      type: String,
      default: 'js-scroller-control-target',
    },
    responsiveTarget: {
      type: String,
      default: 'document',
      validator: (target) => {
        const targets = ['document', 'container'];
        return targets.includes(target);
      },
    },
    forceGrid: Boolean,
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
      this.throttledResetScroller = throttle(() => this.resetScroller(), 50);
      this.throttledScroller = throttle(() => this.updateScrollData(), 50);

      window.addEventListener('resize', this.throttledResetScroller);
      document.addEventListener('scroll', this.throttledScroller);
    },

    resetScroller() {
      if (!this.items.length) {
        return;
      }

      const containerWidth = document.body.clientWidth;

      if (containerWidth !== this.containerWidth) {
        this.containerWidth = containerWidth;

        const responsiveTargetWidth =
          this.responsiveTarget === 'container' ? this.containerWidth : document.body.clientWidth;

        this.breakpoints.forEach((breakpoint) => {
          if (responsiveTargetWidth > breakpoint.documentWidth) {
            this.currentBreakpoint = breakpoint;
          }
        });

        const itemMinTotalWidth = this.currentBreakpoint.itemMinWidth + this.itemSpacing;
        const numItems = (this.numItems = Math.floor((this.containerWidth + this.itemSpacing) / itemMinTotalWidth));

        this.itemWidth = this.containerWidth / numItems - this.itemSpacing + this.itemSpacing / numItems;
        this.itemTotalWidth = this.itemWidth + this.itemSpacing;
       // this.scrollerWidth = this.itemTotalWidth * (this.items.length - 1) + this.itemWidth;
console.log('numitems', numItems)

        this.measuredItems = [];
        let count = 0;

        this.itemHeight  = (808/632) * this.itemWidth;
        this.items.forEach((item, index) => {
          if (index !== 0 ){
            count = index % numItems === 0 ? 0 : count + 1;
          }
         // console.log(count);
          this.measuredItems.push({
            ...item,
            left: this.itemTotalWidth * count,
            top: this.itemHeight * Math.floor(index/numItems) + this.itemSpacing,
            itemIndex: index,
          })
         // console.log(this.measuredItems[index].left)
        });

        // this.$nextTick(() => {
        //   if (this.firstItemEl) {
        //     this.scrollerHeight = 500;//this.firstItemEl.offsetHeight;
        //   }
        // });
      }
    },

    getItemStyle(item) {
      if (this.mounted) {
        return {
          transform: `translate3d(${item.left}px, ${item.top}px, 0)`,
          width: `${this.itemWidth}px`,
        };
      }
    },

    updateScrollData() {
      this.scrollLeft = 0;
      this.scrollTop = window.pageYOffset;
      //console.log( this.scrollTop);
    },

    // prev() {
    //   const cardIndex = Math.round(this.scrollLeft / this.itemTotalWidth);
    //   const left = Math.max(0, cardIndex * this.itemTotalWidth - this.containerWidth - this.itemSpacing);

    //   this.scrollToPosition(left);
    // },

    // next() {
    //   const cardIndex = Math.round((this.scrollLeft + this.containerWidth) / this.itemTotalWidth);
    //   const left = Math.min(this.scrollerWidth - this.containerWidth, cardIndex * this.itemTotalWidth);

    //   this.scrollToPosition(left);
    // },

    scrollToPosition(left) {
      const options = { top: 0, left };

      this.$refs.container.scroll(options);
    },
  },

  computed: {
    firstItemEl() {
      if (this.$refs.items && this.$refs.items.length) {
        return this.$refs.items[0];
      }
      return null;
    },

    controlOffsetStyle() {
      if (this.controlOffset) {
        return {
          top: `${this.controlOffset}px`,
        };
      }
      return '';
    },

    renderedItems() {
      if (this.visibleItems.length && !this.forceGrid) {
        return this.visibleItems;
      }

      if (!this.items.length) {
        return [];
      }

      // if (this.items.length < this.numSsrCards) {
      //   const numItemsMissing = this.numSsrCards - this.items.length;

      //   const emptyPlaceholders = Array(numItemsMissing)
      //     .fill()
      //     .map(() => ({ emptyPlaceholder: true }));

      //   return [...this.items, ...emptyPlaceholders];
      // }

      return this.items.slice(0, this.numSsrCards);
    },

    visibleItems() {
      let startPosition = -1;
      let endPosition = -1;
      let docHeight = window.innerHeight;
      this.measuredItems.forEach((item, index) => {
     //   console.log('this.scrollTop', this.scrollTop);
     //   console.log('item.top', item.top);
        if (startPosition === -1 && item.top > this.scrollTop) {
          startPosition = index;
        }
        if (endPosition === -1 && item.top > this.scrollTop + docHeight) {
          endPosition = index;
        }
        //startPosition = item.top >Math.round(this.scrollTop / this.itemTotalWidth);
      });

      console.log('startPosition', startPosition)
      console.log('endPosition', endPosition)
      //const startPosition = Math.round(this.scrollTop / this.itemTotalWidth);
     // const endPosition = (this.scrollLeft + (this.containerWidth + this.itemSpacing)) / this.itemTotalWidth;
     console.log('docHeight', docHeight)
      const bufferSize = Math.max(docHeight/(this.itemHeight + this.itemSpacing));
      console.log('bufferSize', bufferSize)
      const sliceStart = Math.max(0, startPosition - this.numItems * bufferSize * 2);
      const sliceEnd = Math.min(this.measuredItems.length, endPosition + this.numItems * bufferSize * 2);

console.log('sliceStart', sliceStart)
      console.log('sliceEnd', sliceEnd)
      return this.measuredItems.slice(sliceStart, sliceEnd);
     // return this.measuredItems.slice(0, this.numSsrCards);
    },

    isAtBeginning() {
      return this.scrollLeft === 0;
    },

    isAtEnd() {
      return Math.round(this.scrollLeft) >= Math.round(this.scrollerWidth - this.containerWidth);
    },

    scrollerStyle() {
      if (this.mounted) {
        return { width: `${this.scrollerWidth}px`, height: `${this.scrollerHeight}px` };
      }
      return '';
    },
  },

  mounted() {
    this.mounted = true;

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

:root {
  --content-padding: #{50px};
  --covers-grid-gap: #{20px};
}

$control-size: 40px;

.wrapper {
  position: relative;
}

.container {
  position: relative;
}

.grid {
  display: grid;
  grid-template-columns: repeat(4,1fr);
  grid-template-rows: repeat(5,1fr);

  grid-column-gap: var(--covers-grid-gap);
  grid-row-gap: var(--covers-grid-gap);
  margin: auto;
  padding: 0 var(--content-padding) var(--content-padding);

  &.topPadding {
    padding-top: var(--content-padding);
  }

}

.item {
  display: inline-block;
  left: 0;
  position: absolute;
  top: 0;
}
</style>
