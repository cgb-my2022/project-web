<template>
  <div class='logo'>
    <router-link :to="{name:'dashboard'}">

      <!-- update-begin- author:sunjianlei --- date:20190814 --- for: logo颜色根据主题颜色变化 -->
      <!--      <img v-if="navTheme === 'dark'" src="~@/assets/logo-white.png" alt="logo">-->
      <!--      <img v-else src="~@/assets/logo.png" alt="logo">-->
      <!-- update-begin- author:sunjianlei --- date:20190814 --- for: logo颜色根据主题颜色变化 -->
      <img v-if='collapsed' @click='toggle' src='@/assets/logo.png' alt='' style='width: 66px;height:29px'>
      <img v-else src='@/assets/logo.png' @click='toggle' style='width: 45px;height:20px' alt=''>
      <span style='margin-left:8px;font-size: 14px;font-weight: 400;' v-if='showTitle'>{{ title }}</span>
    </router-link>
  </div>
</template>

<script>
import { mixin } from '@/utils/mixin.js'
import { triggerWindowResizeEvent } from '@/utils/util'
import { mapActions } from 'vuex'


export default {
  name: 'Logo',
  mixins: [mixin],
  props: {
    title: {
      type: String,
      default: '后台系统',
      required: false
    },
    showTitle: {
      type: Boolean,
      default: true,
      required: false
    }
  },
  data() {
    return {
      collapsed: true
    }
  },
  methods: {
    ...mapActions(['setSidebar']),
    toggle() {
      this.collapsed = !this.collapsed
      this.setSidebar(this.collapsed)
      triggerWindowResizeEvent()
    }
  }
}
</script>
<style lang='less' scoped>
/*缩小首页布 局顶部的高度*/
@height: 59px;

.sider {
  box-shadow: none !important;

  .logo {
    height: @height !important;
    line-height: @height !important;
    box-shadow: none !important;
    transition: background 300ms;
    background: #222653;

    a {
      color: white;

      &:hover {
        color: rgba(255, 255, 255, 0.8);
      }
    }
  }

  &.light .logo {
    background-color: #222653;
  }
}
</style>