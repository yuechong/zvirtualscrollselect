<template>
  <a-select
    show-search
    :value="selectValue"
    :filter-option="false"
    :allowClear="allowClear"
    :disabled="disabled"
    :placeholder="placeholder"
    @search="handleSearchFn"
  >
    <a-icon v-if="allowClear" type="close-circle" theme="filled" slot="clearIcon" @click="clearFn" />
    <div slot="dropdownRender" slot-scope="menu">
      <div class="scroll-select-box" :style="scrollBoxStyle" @scroll="handelScrollFn" ref="scrollBox">
        <div class="inner-select-box" :style="{ height: scorllHeght + 'px' }">
          <div class="list-box" :style="`transform:translateY(${offSetY}px);`">
            <div
              class="scroll-select-item"
              v-for="item in list"
              :key="item.index"
              :data-index="item.index"
              :data-value="item.value"
              :style="`height:${itemH}px;line-height:${itemH}px;`"
              :class="{ active: value === item.value }"
              @click="selectChange(item)"
            >
              <a href="javascript:;" :title="item.label">{{ item.label }}</a>
            </div>
          </div>
        </div>
      </div>
    </div>
  </a-select>
</template>

<script>
const debounce = (fn, delay) => {
  let timeoutID = null;
  return function() {
    let context = this;
    clearTimeout(timeoutID);
    timeoutID = setTimeout(() => {
      fn.apply(context, arguments);
    }, delay);
  };
};
export default {
  name: 'ZVirtualScrollSelect',
  props: {
    value: [String, Number, Array],
    options: Array,
    allowClear: { type: Boolean, default: true },
    disabled: { type: Boolean, default: false },
    placeholder: String,
    viewH: { type: Number, default: 200 }, // 可视区域高度
    itemH: { type: Number, default: 40 }, // 列表单行高度
    offSetNum: { type: Number, default: 2 } //上下多渲染2个
  },
  data() {
    return {
      selectValue: undefined,
      searchBool: false,
      filterData: [],
      list: [], // 展示列表
      offSetY: 0, // 动态偏移量
      showNum: 5, // 显示的个数
      handelScrollFn: debounce(this.handelScroll, 10),
      handleSearchFn: debounce(this.handleSearch, 10)
    };
  },
  computed: {
    // 滚动的父容器
    scrollBoxStyle() {
      return { 'overflow-y': 'scroll', 'max-height': this.viewH + 'px', 'min-height': '200px' };
    },
    // 设置总高
    scorllHeght() {
      if (this.searchBool) {
        return this.filterData && this.filterData.length ? this.itemH * this.filterData.length : this.viewH;
      }
      return this.options && this.options.length ? this.itemH * this.options.length : this.viewH;
    }
  },
  watch: {
    value(val) {
      this.valueInit(val);
    },
    options: {
      immediate: true,
      handler(array) {
        if (array && array.length) {
          this.valueInit(this.value);
          // 计算偏量
          this.showNum = Math.ceil(this.viewH / this.itemH) + this.offSetNum;
          this.list = this.options.slice(0, this.showNum);
          this.offSetY = 0;
        }
      }
    }
  },
  methods: {
    /**
     * value 初始化
     */
    valueInit(val) {
      console.log('val', val);
      if (typeof val === 'undefined') {
        this.selectValue = undefined;
      } else {
        if (this.options && this.options.length) {
          let target;
          for (let i = 0; i < this.options.length; i++) {
            const item = this.options[i];
            if (item.value === val) {
              target = item;
              this.selectValue = target.label;
              break;
            }
          }
        }
      }
    },
    clearFn() {
      this.$emit('input', undefined);
      this.$emit('callback', undefined);
    },
    /**
     * 输入框搜索
     */
    handleSearch(value) {
      console.log('value', value);
      console.log(this.selectValue);
      if (typeof value === 'undefined') {
        this.searchBool = false;
      } else {
        this.filterData = this.options.filter(item => item.label.toLowerCase().includes(value.toLowerCase()));
        this.$refs['scrollBox'].scrollTop = 0;
        // 计算偏量
        this.list = this.filterData.slice(0, this.showNum);
        this.offSetY = 0;
        this.searchBool = true;
      }
    },
    /**
     * 选中事件
     */
    selectChange(item) {
      console.log('item', item);
      this.$emit('input', item.value);
      this.$emit('callback', item);
    },
    /**
     * 滚动事件
     */
    handelScroll(e) {
      let scrollTop = e.target.scrollTop;
      // console.log(scrollTop, scrollTop % this.itemH);
      // 滚去的高度除以每项的高度，得出滚去的个数
      let start = Math.floor(scrollTop / this.itemH);
      // console.log('start', start);
      let offset = scrollTop - (scrollTop % this.itemH);
      this.offSetY = offset; //设置动态偏移量模拟滚动

      const options = this.searchBool ? this.filterData : this.options;

      // 当滚动高度大于设置的offsetNum的高度，展示上隐藏内容.滚动到底部区域时，隐藏内容
      const hiddenTopHeight = this.offSetNum * this.itemH;
      if (offset > hiddenTopHeight && offset < this.scorllHeght - this.viewH) {
        this.offSetY = offset - hiddenTopHeight;
        this.list = options.slice(start - this.offSetNum, start + this.showNum); // 根据滚动条高度来截取需要展示的列表区间
      } else {
        this.list = options.slice(start, start + this.showNum); // 根据滚动条高度来截取需要展示的列表区间
      }
    }
  }
};
</script>

<style lang="less">
.scroll-select-box {
  .inner-select-box {
    overflow: hidden;
  }
  .scroll-select-item {
    a {
      display: block;
      color: #ddd;
      overflow: hidden;
      white-space: nowrap;
      text-overflow: ellipsis;
      user-select: none;
      padding: 0px 5px;
      &:hover {
        background: #575f6a;
      }
    }
  }
  .active.scroll-select-item {
    a {
      background: #c73531;
    }
  }
}
</style>
