<template>
  <div>
    <!-- 一条文章单元格 -->
    <van-cell>
      <!-- 标题区域的插槽 -->
      <template #title>
        <div class="title-box">
          <!-- 标题 -->
          <span>{{ artObj.title }}</span>
          <!-- 单图 -->
          <!-- <img
            v-if="artObj.cover.type === 1"
            class="thumb"
            :src="artObj.cover.images[0]"
            alt=""
          /> -->
          <van-image v-if="artObj.cover.type === 1"
            class="thumb"
            :src="artObj.cover.images[0]"
            alt="">
            <template v-slot:error>图片走丢了</template>
          </van-image>
        </div>
        <!-- 多图 -->
        <div class="thumb-box" v-if="artObj.cover.type > 1">
          <!-- <img
            v-for="(imgUrl, index) in artObj.cover.images"
            :key="index"
            class="thumb"
            :src="imgUrl"
            alt=""
          /> -->
          <van-image v-for="(imgUrl, index) in artObj.cover.images"
            :key="index"
            class="thumb"
            :src="imgUrl"
            alt="">
            <template v-slot:error>图片走丢了</template>
          </van-image>
        </div>
      </template>
      <!-- label 区域的插槽 -->
      <template #label>
        <div class="label-box">
          <div>
            <span>{{ artObj.aut_name }}</span>
            <span>{{ artObj.comm_count }}评论</span>
            <span>{{ formatTime(artObj.pubdate) }}</span>
          </div>
          <!-- 反馈按钮 -->
          <van-icon name="cross" @click.stop="show = true" v-if="isShow" />
        </div>
      </template>
    </van-cell>
    <!-- 反馈面板 -->
    <van-action-sheet
      v-model="show"
      :actions="actions"
      @select="onSelect"
      :cancel-text="bottomText"
      @cancel="cancelFn"
      @close="closeFn"
      get-container="body"
    />
  </div>
</template>

<script>
// 目标1：点击"反馈垃圾内容"数据的切换
// 1. 监听点击事件，区分用户点击的是哪一个
// 2. 切换actions数组里的数据

// 目标2：二级反馈数据出现，取消按钮的文字要改成"返回"
// :cancel-text="bottomText" 设置变量，onSelect中，把变量值换成"返回"

// 目标3：点击底部按钮，要区分判断
// 1. 标签@cancel取消事件和事件方法
// 2. 在里面用bottomText的值，判断用户点击的是否为"返回"
// 把actions数据源换成firstActions
// 把bottomText文字换成"取消"
// 强制让反馈面板show=true留下
import { timeAgo } from '@/utils/date.js'
import { firstActions, secondActions } from '@/api/report.js'
export default {
  props: {
    artObj: Object, // 文章对象
    isShow: {
      type: Boolean,
      default: true // 例如：首页
    }
  },
  data () {
    // 内部的局部变量
    return {
      show: false, // 反馈面板显示/隐藏
      actions: firstActions, // 反馈面板选项数组(套的是对象)
      bottomText: '取消' // 底部按钮的文字
    }
  },
  methods: {
    formatTime: timeAgo, // formatTime就是函数
    // 反馈面板-"选项"选择事件
    onSelect (action, item) {
      // console.log(action) // {name: '反馈垃圾内容'} action-反馈选项的对象
      // console.log(item) // 索引值
      // 默认情况下点击选项时不会自动收起
      // 可以通过 close-on-click-action 属性开启自动收起
      // this.show = false

      if (action.name === '反馈垃圾内容') {
        this.actions = secondActions
        this.bottomText = '返回'
      } else if (action.name === '不感兴趣') {
        this.$emit('disLikeEV', this.artObj.art_id) // 子向父传
        this.show = false // 无论成功/失败-直接让反馈面板隐藏
      } else {
        // 二级反馈选项
        this.$emit('reportEV', this.artObj.art_id, action.value)
        this.show = false // 关闭-下面的cancel会把反馈选项切回来
      }
    },
    // 反馈面板-底部按钮-点击事件
    cancelFn () {
      if (this.bottomText === '返回') {
        this.show = true // 强制让它回来
        this.actions = firstActions
        this.bottomText = '取消'
      }
    },
    // 反馈面板-关闭面板-触发事件
    closeFn () {
      this.actions = firstActions
      this.bottomText = '取消'
    }
    // 文章单元格-点击事件 <van-cell @click="cellClickFn">
    // cellClickFn () {
    //   console.log(123)
    // }
  }
}
</script>

<style scoped lang="less">
/* 标题样式 */
.title-box {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
}

/* label描述样式 */
.label-box {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

/* 文章信息span */
.label-box span {
  margin: 0 3px;
  &:first-child {
    margin-left: 0;
  }
}

/* 图片的样式, 矩形黄金比例：0.618 */
.thumb {
  width: 113px;
  height: 70px;
  background-color: #f8f8f8;
  object-fit: cover;
}

/* 三图, 图片容器 */
.thumb-box {
  display: flex;
  justify-content: space-between;
}
</style>
