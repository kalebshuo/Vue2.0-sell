<template>
<transition name="move">
  <div v-show="showFlag" class="food" ref="food">
    <div class="food-content">
        <div class="image-header">
            <img :src="food.image">
            <div class="back" @click="hide">
                <i class="icon-arrow_lift"></i>
            </div>
        </div>
        <div class="content">
            <h1 class="title">{{food.name}}</h1>
            <div class="detail">
                <span class="sell-count">月售{{food.sellCount}}份</span>
                <span class="rating">好评率{{food.rating}}</span>
            </div>
             <div class="price">
                <span class="now">¥{{food.price}}</span>
                <span class="old" v-show="food.oldPrice">¥{{food.oldPrice}}</span>
            </div>
            <div class="cartcontrol-wrapper">
                <cartcontrol :food="food"></cartcontrol>
            </div>
            <transition name="fade">
                <div @click="addFirst" class="buy" v-show="!food.count || food.count===0">加入购物车</div>
            </transition>
        </div>
        <split  v-show="food.info"></split>
        <div class="info" v-show="food.info">
            <h1 class="title">商品信息</h1>
            <p class="text">{{food.info}}</p>
        </div>
        <split  v-show="food.info"></split>
        <div class="rating">
            <h1 class="title">商品评价</h1>
            <ratingselect :select-type="selectType" :only-content="onlyContent" :desc="desc" :ratings="food.ratings" @select="selectRating" @toggle="toggleContent"></ratingselect>
            <div class="rating-wrapper">
                <ul v-show="food.ratings&&food.ratings.length">
                    <li v-show="needShow(rating.rateType,rating.text)" v-for="(rating,index) in food.ratings" class="rating-item border-1px" :key="index">
                        <div class="user">
                            <span class="name">{{rating.username}}</span>
                            <img class="avatar" width="12" height="12" :src="rating.avatar">
                        </div>
                        <div class="time">{{rating.rateTime | formatDate}}</div>
                        <p class="text"><span :class="{'icon-thumb_up':rating.rateType===0,'icon-thumb_down':rating.rateType===1}"></span>{{rating.text}}</p>
                </li> <!--使用对象的多个样式绑定-->
            </ul>
            <div class="no-rating" v-show="!food.ratings|| !food.ratings.length">暂无评价</div>
        </div>
        </div>
    </div>
  </div>
</transition>
</template>

<script type='text/ecmascript-6'>
import BScroll from 'better-scroll';
import cartcontrol from 'components/cartcontrol/cartcontrol';
import Vue from 'vue';
import split from 'components/split/split';
import ratingselect from 'components/ratingselect/ratingselect';
import {formatDate} from 'common/js/date';
const All = 2;
export default {
    props: {
        food: {
            type: Object
        }
    },
    data() {
        return {
            showFlag: false,
            selectType: All,
            onlyContent: true,
            desc: {
                all: '全部',
                positive: '满意',
                negative: '吐槽'
            }
        };
    },
    methods: {
        show() {
            this.showFlag = true;
            this.$nextTick(() => {
                    if (!this.scroll) {
                   this.scroll = new BScroll(this.$refs.food, {
                       click: true
                   });
                    } else {
                        this.scroll.refresh();
                    }
                });
        },
        hide() {
            this.showFlag = false;
        },
        addFirst(event) {
            if (!event._constructed) {  // 阻止浏览器原生点击
                 return;
             }
            Vue.set(this.food, 'count', 1);
        },
        needShow(type, text) {
            if (this.onlyContent && !text) {
                return false;
            }
            if (this.selectType === All) {
                return true;
            } else {
                return this.selectType === type;
            }
        },
        selectRating(type) {
            this.selectType = type;
            this.$nextTick(() => {
                this.scroll.refresh(); // dom更新是异步的，所以不能直接刷新scroll，需要等dom更新之后再刷新滚动条。
            });
        },
        toggleContent() {
            this.onlyContent = !this.onlyContent;
            this.$nextTick(() => {
                this.scroll.refresh();
            });
        }
    },
    filters: {
      formatDate(time) {
        let date = new Date(time);
        return formatDate(date, 'yyyy-MM-dd hh:mm');
      }
    },
    components: {
        cartcontrol,
        split,
        ratingselect
    }
};
</script>

<style lang='stylus' rel='stylesheet/stylus'>
@import "../../common/stylus/mixin";
    .food
        position fixed 
        top 0
        left 0
        bottom 48px
        z-index 30
        width 100%
        background #fff
        transform translated3d(0,0,0)
        &.move-enter-active,&.move-leave-active
            transition all 0.2s linear  
        &.move-enter,&.move-leave-active
            transform translate3d(100%,0,0)
        .image-header
            position relative 
            top 0
            left 0
            width 100%
            height 0
            padding-top 100%  // padding-**的百分比参照物是父元素的宽度，所以可以利用padding-top撑开高度，占位用，不至于图片加载不出来导致布局难看
            img 
                position absolute // 图片默认不会占padding所以需要定位
                top 0
                left 0
                height 100%
                width 100%
            .back
                position absolute
                top 10px
                left 0
                .icon-arrow_lift
                    display block 
                    padding 10px
                    font-size 20px
                    color #fff
        .content
            position relative
            padding 18px
            font-size 0
            .title
                margin-bottom 8px
                font-size 14px
                font-weight 700
                color rgb(7,17,27)
                line-height 14px
            .detail
                color rgb(147,153,159)
                line-height 10px
                height 10px
                margin-bottom 18px
                .sell-count
                    font-size 10px
                    margin-right 12px
                .rating
                    font-size 10px
            .price
                line-height 24px
                font-weight 700
                .now
                    margin-right 8px
                    font-size 14px
                    color rgb(240,20,20)
                .old
                    font-size 14px
                    color rgb(147,153,159)
                    text-decoration line-through
            .cartcontrol-wrapper
                position absolute
                bottom 12px
                right 12px
            .buy
                position absolute
                bottom 18px
                right 18px
                z-index 10
                width 74px
                height 24px
                line-height 24px
                border-radius 12px
                background-color rgb(0,160,220)
                text-align center
                font-size 10px
                color #fff
                opacity 1
                &.fade-enter-active,&.fade-leave-active
                    transition 1s all
                &.fade-enter,&.fade-leave-active // v-enter：定义进入过渡的开始状态。在元素被插入时生效，在下一个帧移除
                    opacity 0
                    z-index: -1
        .info
            padding 18px
            .title
                font-size 14px
                color rgb(7,17,27)
                margin-bottom 6px
                line-height 14px
            .text
                padding 0 8px
                font-size 12px
                color rgb(77,85,93)
                line-height 24px
        .rating
            padding-top 18px
            .title
                font-size 14px
                color rgb(7,17,27)
                margin-left 18px
                line-height 14px
            .rating-wrapper
                padding 0 18px
                font-size 0
                .rating-item
                    padding 16px 0
                    position relative
                    border-1px(rgba(7,17,27,0.1))
                    .user
                        position absolute
                        top 16px
                        right 0
                        line-height 12px
                        .name
                            font-size 10px
                            color rgb(147,153,159)
                            margin-right 6px
                            vertical-align top
                        .avatar
                            border-radius 50%
                    .time
                        margin-bottom 6px // css规范：影响布局的写在前面
                        font-size 10px
                        color rgb(147,153,159)
                        line-height 12px
                    .text
                        font-size 12px
                        color rgb(7,17,27)
                        line-height 16px
                        .icon-thumb_up,        .icon-thumb_down
                            margin-right 4px
                            color rgb(147,153,159)
                            line-height 24px
                            font-size 12px
                        .icon-thumb_up
                            color rgb(0,160,220)
            .no-rating
                padding: 16px 0
                font-size: 12px
                color: rgb(147, 153, 159)


                     
</style>