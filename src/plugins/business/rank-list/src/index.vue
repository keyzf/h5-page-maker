<template>
  <div class="qk-rank-list">
    <div
      v-if="list.length === 0"
      :class="cx('empty')"
    >
      <!-- <img class="empty-img" :src="imageSrc_emptyBg" alt=""> -->
    </div>
    <ul
      v-if="list.length > 0"
      :class="cx('list-wrapper')"
    >
      <li
        :class="cx('item')"
        :style="{color:color_title}"
      >
        <div :class="cx('item-title')">{{ text_rank }}</div>
        <div :class="cx('item-title')">{{ text_avatar }}</div>
        <div :class="cx('item-title')">{{ text_nick }}</div>
        <div :class="cx('item-title')">{{ text_giftCount }}</div>
      </li>
      <li
        v-for="(item, index) in list"
        :class="cx('item')"
        :key="index"
      >
        <div :class="cx('item-rank')">
          <template v-if="index===0">
            <img :src="imageSrc_firstIcon" />
          </template>
          <template v-else-if="index===1">
            <img :src="imageSrc_secondIcon" />
          </template>
          <template v-else-if="index===2">
            <img :src="imageSrc_thirdIcon" />
          </template>
          <template v-else>{{ index + 1 }}</template>
        </div>
        <div :class="cx('item-avatar')">
          <img
            :src="item.thumb"
            alt=""
          />
        </div>
        <div :class="cx('item-nick')">
          <div>{{ item.name }}</div>
          <div
            class="remark"
            v-if="index===0"
          >{{ text_firstRemark }}</div>
          <div
            class="remark"
            v-if="index===1"
          >{{ text_secondRemark }}</div>
          <div
            class="remark"
            v-if="index===2"
          >{{ text_thirdRemark }}</div>
        </div>
        <div :class="cx('item-score')">
          <img
            v-if="imageSrc_giftIcon"
            :class="cx('item-icon')"
            :src="imageSrc_giftIcon"
            alt=""
          />
          <span class="item-total"><template v-if="imageSrc_giftIcon">x</template> {{ item.total || 0 }}</span>
        </div>
      </li>
    </ul>
    <my-rank
      ref="myRank"
      :user-info="userInfo"
      :color-my-rank="color_myRank"
      :image-src-my-rank="imageSrc_myRank"
      :my-rank-bottom="number_myRankBottom"
      :image-src-gift-icon="imageSrc_giftIcon"
    />
  </div>
</template>

<script>
import API from "./apis";
import MyRank from "./components/my-rank";
import { demoList, demoUserInfo } from "./mock";

const classPrefix = "qk-rank-list";
const RANK_MAP = { 0: "第一名", 1: "第二名", "-1": "未上榜" };

export default {
  name: "QkRankList",
  components: { MyRank },
  props: {
    configBtn: {
      label: "数据源配置",
      type: String,
      default: `/pages/activity/rank-config/index.html`,
    },
    color_title: {
      label: "标题颜色",
      type: String,
      default: "#999999",
    },
    color_myRank: {
      label: "我的排名颜色",
      type: String,
      default: "#FFF",
    },
    imageSrc_myRank: {
      label: "我的排名背景",
      type: String,
      default: "//img.ikstatic.cn/MTU5MDEzMzA5NTg1MyM2NDQjcG5n.png",
    },
    number_myRankBottom: {
      label: "我的排名bottom值",
      type: Number,
      default: 10,
    },
    text_rank: {
      label: "排名",
      type: String,
      default: "排名",
    },
    text_avatar: {
      label: "头像",
      type: String,
      default: "头像",
    },
    text_nick: {
      label: "昵称",
      type: String,
      default: "昵称",
    },
    text_giftCount: {
      label: "礼物数量",
      type: String,
      default: "礼物数量",
    },
    imageSrc_giftIcon: {
      label: "礼物图标",
      type: String,
      default: "//img.ikstatic.cn/MTU4OTg2ODMxNjUyMyMyMTYjcG5n.png",
    },
    imageSrc_firstIcon: {
      label: "第一名图标",
      type: String,
      default: "//img.ikstatic.cn/MTU4OTg2ODM2Mjg0OSM4MjQjcG5n.png",
    },

    imageSrc_secondIcon: {
      label: "第二名图标",
      type: String,
      default: "//img.ikstatic.cn/MTU4OTg2ODMzNzMzOCM2NTkjcG5n.png",
    },
    imageSrc_thirdIcon: {
      label: "第三名图标",
      type: String,
      default: "//img.ikstatic.cn/MTU4OTg2ODM1MjU0NyM4MzMjcG5n.png",
    },
    text_firstRemark: {
      label: "第一名备注文字",
      type: String,
      default: "奖励：xxxx元，夏日1头像框",
    },
    text_secondRemark: {
      label: "第二名备注文字",
      type: String,
      default: "奖励：xxxx元，夏日2头像框",
    },
    text_thirdRemark: {
      label: "第三名备注文字",
      type: String,
      default: "奖励：xxxx元，夏日3头像框",
    },
    imageSrc_emptyBg: {
      label: "省缺图",
      type: String,
      default: "//img.ikstatic.cn/MTU3OTE4MTYwMzIzMSMzMzcjcG5n.png",
    },
    currentPageIndex: {
      type: Number,
      default: 0,
    },
  },
  data() {
    return {
      RANK_MAP,
      allList: [], // 所有排行榜到数据
      allUserInfo: [], // 所有自己到数据
    };
  },
  computed: {
    list() {
      return this.allList[this.currentPageIndex] || [];
    },
    userInfo() {
      return (
        this.allUserInfo[this.currentPageIndex] || {
          nick: "", //昵称
          portrait: "", //头像
          total: 0, //数值
          rank: "", //排名，0为第一名，1为第二名，-1表示没上榜
        }
      );
    },
  },
  mounted() {
    this.fetchList();
  },
  methods: {
    cx(s) {
      return `${classPrefix}-${s}`;
    },
    fetchList() {
      API.GET_RANK_DATA(this.globalVariable).then((res) => {
        if (Array.isArray(res.data.list) && res.data.list.length > 0) {
          this.allList = res.data.list || [];
          this.allUserInfo = res.data.user_info || [];
        } else if (this.isPreview) {
          // 模拟一些数据
          this.allList = [demoList, demoList, demoList];
          this.allUserInfo = [demoUserInfo, demoUserInfo, demoUserInfo];
        }
      });
    },
  },
};
</script>

<style lang="scss" scoped>
.qk-rank-list {
  &-item {
    margin-bottom: 12px;
    padding: 0 12px;
    display: flex;
    align-items: center;

    .remark {
      font-size: 11px;
      margin-top: 2px;
      line-height: 1;
      opacity: 0.6;
    }

    &-title {
      &:nth-of-type(1) {
        width: 10%;
        text-align: center;
      }

      &:nth-of-type(2) {
        width: 25%;
        text-align: center;
      }

      &:nth-of-type(3) {
        flex: 1;
        // text-align: left;
        overflow: hidden;
        white-space: nowrap;
        text-overflow: ellipsis;
      }

      &:nth-of-type(4) {
        text-align: right;
      }
    }

    &-rank {
      width: 10%;
      font-weight: 500;
      text-align: center;

      img {
        display: inline-block;
        width: 100%;
      }
    }

    &-avatar {
      width: 25%;

      img {
        display: block;
        width: 40px;
        height: 40px;
        margin: auto;
        border-radius: 50%;
        overflow: hidden;
        // background-color: #d9d9d9;
      }
    }

    &-nick {
      flex: 1;
      overflow: hidden;
      margin-right: 5px;

      > div {
        overflow: hidden;
        white-space: nowrap;
        text-overflow: ellipsis;
      }
    }

    &-score {
      display: flex;
      text-align: right;
      align-items: center;
      justify-content: flex-end;
      overflow: hidden;
      .item-total {
        max-width: 100%;
        overflow: hidden;
        white-space: nowrap;
        text-overflow: ellipsis;
      }
    }

    &-icon {
      width: 34px;
      height: 34px;
      display: inline-block;
    }
  }

  &-empty {
    height: 400px;
    display: flex;
    align-items: center;
    justify-content: center;

    .empty-img {
      width: 200px;
    }
  }
}
</style>
