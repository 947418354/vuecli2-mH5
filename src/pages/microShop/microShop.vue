<template>
  <div class="page p-micro-shop">
    <div class="p-title">微店</div>
    <!-- 上拉加载 下拉刷新的组件.不要在弹性项目中使用它 因为无法准确获取mescroll高度 导致下拉刷新bug -->
    <mescroll-vue
      ref="mescroll"
      :down="mescrollDown"
      :up="mescrollUp"
      class="scroll-cont"
      @init="mescrollInit"
    >
      <!-- 带tabs产品列表 -->
      <tab active-color="#1D96EB" bar-active-color="#1D96EB" class="tab-container vux-1px-b">
        <tab-item
          v-for="(item, index) in tabList"
          :key="index"
          :ref="item.value"
          :selected="item.value === activeTab"
          @on-item-click="onTabClick"
        >{{ item.label }}</tab-item>
      </tab>
      <div class="product-list">
        <swipeout>
          <draggable
            v-model="productList"
            :touch-start-threshold="5"
            :support-pointer="false"
            :delay-on-touch-only="true"
            :delay="2000"
            @start="drag = true"
            @end="drag = false"
          >
            <transition-group type="transition">
              <div v-for="(item, i) in productList" :key="item.id">
                <swipeout-item transition-mode="follow">
                  <div slot="right-menu">
                    <swipeout-button
                      type="primary"
                      class="delete-icon"
                      @click.native="deleteProduct(item)"
                    />
                  </div>
                  <div slot="content">
                    <div>{{item.name}}</div>
                  </div>
                </swipeout-item>
              </div>
            </transition-group>
          </draggable>
        </swipeout>
      </div>
    </mescroll-vue>
    <jiuDialog></jiuDialog>
  </div>
</template>

<script>
import { find, isNumber } from "lodash";
import jiuDialog from '@/components/dialog/jiuDialog/jiuDialog'
import {
  Tab,
  TabItem,
  XDialog,
  Swipeout,
  SwipeoutItem,
  SwipeoutButton,
  TransferDomDirective as TransferDom
} from "vux";
import draggable from "vuedraggable";
import MescrollVue from "mescroll.js/mescroll";
export default {
  data() {
    return {
      mescroll: null,
      mescrollDown: {
        // isLock: true,
        callback: this.mescrollDownCallBack,
      },
      mescrollUp: {
        //上拉配置
        callback: this.upCallback,
        noMoreSize: 1,
        page: {
          size: 5
        },
        empty: {
          // 列表第一页无任何数据时,显示的空提示布局; 需配置warpId才生效;
          warpId: "nodata", // 父布局的id;
          // icon: './static/mescroll/mescroll-empty.png', // 图标,支持网络图
          tip: "暂无相关数据~" // 提示
        },
        htmlNodata: '<p class="upwarp-nodata">-- 没有更多数据了 --</p>'
      },
      tabList: [
        // tabs的数据
        {
          value: "1",
          label: "tab-itema"
        },
        {
          value: "2",
          label: "tab-itemb"
        },
        {
          value: "3",
          label: "tab-itemc"
        }
      ],
      activeTab: "1",
      productList: [
        {
          id: "0",
          name: "产品名称"
        },
        {
          id: "-1",
          name: "产品名称"
        },
      ]
    };
  },
  components: {
    Tab,
    TabItem,
    XDialog,
    Swipeout,
    SwipeoutItem,
    SwipeoutButton,
    draggable,
    MescrollVue,
    jiuDialog,
  },
  methods: {
    mescrollInit(mescroll) {
      this.mescroll = mescroll;
    },
    // 下拉回调
    mescrollDownCallBack(mescroll) {
      // 下拉刷新
      mescroll.resetUpScroll()
    },
    upCallback(page, mescroll) {
      console.log("上拉回调");
      if (page.num === 1) {
        this.productList = [];
      }
      setTimeout(() => {
        const list = [];
        for (let i = 0; i < page.size; i++) {
          list.push({
            id: page.num * page.size + i + "",
            name: "产品名称"
          });
        }
        const resultContent = {
          rows: list,
          total: 50
        };
        const { rows, total } = resultContent;
        this.productList = this.productList.concat(rows || []);
        if (isNumber(rows.length)) {
          this.$nextTick(() => {
            if (this.productList.length === total) {
              mescroll.endBySize(rows.length, total);
            } else {
              mescroll.endSuccess(rows.length);
            }
          });
        }
      }, 2000);
      /*request
        .queryMsProduct()
        .then(res => {
          if (res.data.resultFlag) {
            const { rows, total } = res.data.resultContent;
            this.productList = this.productList.concat(rows || []);
            if (isNumber(rows.length)) {
              this.$nextTick(() => {
                if (this.productList.length === total) {
                  mescroll.endBySize(rows.length, total);
                } else {
                  mescroll.endSuccess(rows.length);
                }
              });
            }
          } else {
            mescroll.endSuccess(0);
          }
        })
        .catch(err => {
          console.log("发生错误：", err);
        });*/
    },
    onTabClick(id) {
      const targetTabItem = find(this.tabList, (item, index) => index === id);

      if (targetTabItem.value === this.activeTab) return;
      // this.setActiveTab(targetTabItem.value);
      this.activeTab = targetTabItem.value;
      this.mescroll.resetUpScroll();
    }
  }
};
</script>

<style lang="less">
.p-micro-shop {
  width: 100%;
  height: 100%;
  .scroll-cont {
    background: #060;
  }
}
</style>
