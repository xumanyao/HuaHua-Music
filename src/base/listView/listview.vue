<template>
  <scroll class="listview"
          :data="data"
          ref="listView"
          :listenScroll="listenScroll"
          @scroll='scroll'
          :probe-type="probeType">
    <ul>
      <li v-for="group in data" ref="listGroup" class="list-group">
        <h2 class="list-group-title">{{group.title}}</h2>
        <ul>
          <li v-for="item in group.items" class="list-group-item" @click="selectItem(item)">
            <img v-lazy="item.avatar" class="avatar">
            <span class="name">{{item.name}}</span>
          </li>
        </ul>
      </li>
    </ul>
    <div class="list-shortcut" @touchstart="onShortcutTouchStart" @touchmove.stop.prevent="onShortcutTouchMove">
      <ul>
        <li v-for="(item,index) in shortcutIndex"
            class="item"
            :data-index="index"
            :class="{'current':currentIndex === index}"
        >{{item}}
        </li>
      </ul>
    </div>
    <div class="list-fixed" ref="fixed" v-show="fixedTitle">
      <div class="fixed-title">{{fixedTitle}}</div>
    </div>
    <div class="loading-container" v-show="!data.length">
      <loading></loading>
    </div>
  </scroll>

</template>

<script>
  import Scroll from 'base/scroll/scroll';
  import {getData} from 'common/js/dom';
  import Loading from 'base/loading/loading';

  const ANCHOR_HEIGHT = 18;
  const TIELE_HEIGHT = 30;
  export default {
    name: 'listview',
    created() {
      this.touch = {};
      this.listenScroll = true;
      this.listHeight = [];
      this.probeType = 3;
    },
    components: {
      Scroll,
      Loading
    },
    computed: {
      shortcutIndex() {
        return this.data.map((group) => {
          return group.title.substr(0, 1);
        });
      },
      fixedTitle() {
        if (this.scrollY > 0) {
          return '';
        }
        return this.data[this.currentIndex] ? this.data[this.currentIndex].title : '';
      }
    },
    props: {
      data: {
        type: Array,
        default: []
      }
    },
    data() {
      return {
        scrollY: -1,
        currentIndex: 0,
        diff: -1
      };
    },
    watch: {
      data() {
        setTimeout(() => {
          this._calculateHeight();
        }, 20);
      },
      scrollY(newY) {
        const listHeight = this.listHeight;
        if (newY > 0) {
          this.currentIndex = 0;
          return;
        }
        for (let i = 0; i < listHeight.length - 1; i++) {
          let heigh1 = listHeight[i];
          let heigh2 = listHeight[i + 1];
          if (-newY >= heigh1 && -newY < heigh2) {
            this.currentIndex = i;
            this.diff = heigh2 + newY;
            return;
          }
        }
        this.currentIndex = listHeight.length - 2;
      },
      diff(newVal) {
        let fixedTop = (newVal > 0 && newVal < TIELE_HEIGHT) ? newVal - TIELE_HEIGHT : 0;
        if (this.fixedTop === fixedTop) {
          return;
        }
        this.fixedTop = fixedTop;
        this.$refs.fixed.style.transform = `translate3d(0,${fixedTop}px,0)`;
      }
    },
    methods: {
      selectItem(item) {
        this.$emit('selectItem', item);
      },
      onShortcutTouchStart(e) {
        let anchorIndex = getData(e.target, 'index');
        let firstTouch = e.touches[0];
        this.touch.pageY1 = firstTouch.pageY;
        this.touch.anchorIndex = anchorIndex;
        this._scrollTo(anchorIndex);
      },
      onShortcutTouchMove(e) {
        let firstTouch = e.touches[0];
        this.touch.pageY2 = firstTouch.pageY;
        let dalta = (this.touch.pageY2 - this.touch.pageY1) / ANCHOR_HEIGHT | 0;
        let anchorIndex = dalta + parseInt(this.touch.anchorIndex);
        this._scrollTo(anchorIndex);
      },
      scroll(pos) {
        this.scrollY = pos.y;
      },
      _scrollTo(index) {
        if (!index && index !== 0) {
          return;
        }
        if (index < 0) {
          index = 0;
        } else if (index > this.listHeight.length - 2) {
          index = this.listHeight.length;
        }
        this.scrollY = -this.listHeight[index];
        this.$refs.listView.scrollToElement(this.$refs.listGroup[index], 0);
      },
      _calculateHeight() {
        this.listHeight = [];
        const list = this.$refs.listGroup;
        let height = 0;
        this.listHeight.push(height);
        for (let i = 0; i < list.length; i++) {
          let curHeight = list[i].clientHeight;
          height += curHeight;
          this.listHeight.push(height);
        }
      }
    }
  };
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"

  .listview
    position: relative
    width: 100%
    height: 100%
    overflow: hidden
    background: $color-background
    .list-group
      padding-bottom: 30px
      .list-group-title
        height: 30px
        line-height: 30px
        padding-left: 20px
        font-size: $font-size-small
        color: $color-text-l
        background: $color-highlight-background
      .list-group-item
        display: flex
        align-items: center
        padding: 20px 0 0 30px
        .avatar
          width: 50px
          height: 50px
          border-radius: 50%
        .name
          margin-left: 20px
          color: $color-text-l
          font-size: $font-size-medium
    .list-shortcut
      position: absolute
      z-index: 30
      right: 0
      top: 50%
      transform: translateY(-50%)
      width: 20px
      padding: 20px 0
      border-radius: 10px
      text-align: center
      background: $color-background-d
      font-family: Helvetica
      .item
        padding: 3px
        line-height: 1
        color: $color-text-l
        font-size: $font-size-small
        &.current
          color: $color-theme
    .list-fixed
      position: absolute
      top: 0
      left: 0
      width: 100%
      .fixed-title
        height: 30px
        line-height: 30px
        padding-left: 20px
        font-size: $font-size-small
        color: $color-text-l
        background: $color-highlight-background
    .loading-container
      position: absolute
      width: 100%
      top: 50%
      transform: translateY(-50%)
</style>
