<template>
  <div id="home"> 
    <nav-bar class="home-nav">
      <div slot="center">购物车</div>
    </nav-bar>

    <tab-control :title="['流行', '新款', '精选']" 
    @tabClick='tabClick' ref="tabControl1"
    class="tab-control" v-show="isTabFixed"/>
    
    <scroll class="content" 
      ref="scroll" 
      :probe-type="3" 
      @scroll="contentScroll"
      :pullUpLoad="true"
      @pullingUp="loadMore">
      <home-swiper :banners='banners' @HomeSwiper="HomeSwiper"></home-swiper>
      <recommend-view :recommends='recommends'/>
      <feature-view/>
      <tab-control :title="['流行', '新款', '精选']" 
      @tabClick='tabClick' ref="tabControl2"/>
      <goods-list :goods='showGoods' />
    </scroll>
    <back-top @click.native="backClick" v-show="isShow"/>
    
  </div>
</template>

<script>
import HomeSwiper from './ChildComps/HomeSwiper'
import RecommendView from './ChildComps/RecommendView'
import FeatureView from './ChildComps/FeatureView'
import GoodsList from 'components/content/goods/GoodsList'

import NavBar from 'components/common/navbar/NavBar'
import TabControl from 'components/content/TabControl'
import Scroll from 'components/common/scroll/Scroll'
import BackTop from 'components/content/backTop/BackTop.vue'

import {debounce} from 'common/utiles.js'
import {getHomeMultidata, getHomeGoods} from 'network/home'


export default {
  name: 'Home',

  components: {
    HomeSwiper,
    RecommendView,
    FeatureView,
    NavBar,
    TabControl,
    GoodsList,
    Scroll,
    BackTop
  },

  data(){
    return {
      banners: [],
      recommends: [],
      goods: {
        'pop': {page: 0, list: []},
        'new': {page: 0, list: []},
        'sell': {page: 0, list: []}
      },
      currentType: 'pop',
      isShow: false,
      tabOffsetTop: 0,
      isTabFixed: false,
      saveY: 0
    }
  },

  created() {
    //1.请求轮播图及推荐数据
    this.getHomeMultidata()
    //2.请求商品数据
    this.getHomeGoods('pop')
    this.getHomeGoods('new')
    this.getHomeGoods('sell')
  },

  mounted() {
    //1.图片加载完成的事件监听
    const refresh = debounce(this.$refs.scroll.refresh,200)
    //监听item中图片加载完成
    this.$bus.$on('imgLoad', () => {
      refresh()
    })
  },

  computed: {
    showGoods() {
      return this.goods[this.currentType].list
    }
  },

  methods: {
    /**
     * 普通方法
     */
    tabClick(index) {
      switch (index) {
        case 0: 
          this.currentType = 'pop'
          break
        case 1:
          this.currentType = 'new'
          break
        case 2:
          this.currentType = 'sell'
          break
      }
      this.$refs.tabControl1.currentType = index;
      this.$refs.tabControl2.currentType = index
    },
    backClick() {
      this.$refs.scroll.scrollTo(0, 0, 500)
    },
    contentScroll(poistion) {
      //1.判断backTop是否显示
      this.isShow = (-poistion.y) > 1000
      //2.判断tabControl是否吸顶
      this.isTabFixed = (-poistion.y) > this.tabOffsetTop
    },
    loadMore() {
      this.getHomeGoods(this.currentType)
      this.$refs.scroll.finishPullUp()
      //实时刷新
      this.$refs.scroll.scroll.refresh()
    },
    HomeSwiper() {
      this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop
    },
    activated() {
      this.$refs.scroll.scrollTo(0, this.saveY, 0)
      this.$refs.scroll.refresh()
    },
    deactivated() {
      this.saveY = this.$refs.scroll.getScrollY()
    },


    /**
     * 网络请求相关的方法
     */
    getHomeMultidata() {
      getHomeMultidata().then(res => {
        this.banners = res.data.banner.list;
        this.recommends = res.data.recommend.list
      })
    },
    getHomeGoods(type) {
      const page = this.goods[type].page + 1
      getHomeGoods(type, page).then(res => {
        this.goods[type].list.push(...res.data.list)
      })
      this.goods[type].page += 1

    }
  }
}
</script>

<style scoped>
  #home {
    padding-top: 44px;
    height: 100vh;
    position: relative;
  }

  .home-nav {
    background-color: var(--color-tint);
    color: #fff;
    position: fixed;
    left: 0;
    right: 0;
    top: 0;
    z-index: 9;
  }


  .content {
    overflow: hidden;
    position: absolute;
    top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;
  }

  .tab-control {
    position: relative;
    z-index: 9;
  }
</style>