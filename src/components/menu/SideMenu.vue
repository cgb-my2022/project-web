<template>
  <a-layout-sider
    :class="['sider', isDesktop() ? null : 'shadow', theme, fixSiderbar ? 'ant-fixed-sidemenu' : null ]"
    width='200px'
    :collapsible='collapsible'
    v-model='collapsed'
    :trigger='null'>
    <logo />
    <div class='slider-menu'>
      <s-menu
        :collapsed='collapsed'
        :menu='menus'
        :theme='theme'
        @select='onSelect'
        @click='onClick'
        :mode='mode'
        :style='smenuStyle'>
      </s-menu>
    </div>
  </a-layout-sider>

</template>

<script>
import ALayoutSider from 'ant-design-vue/es/layout/Sider'
import Logo from '../tools/Logo'
import SMenu from './index'
import { mixin, mixinDevice } from '@/utils/mixin.js'

export default {
  name: 'SideMenu',
  components: { ALayoutSider, Logo, SMenu },
  mixins: [mixin, mixinDevice],
  props: {
    mode: {
      type: String,
      required: false,
      default: 'inline'
    },
    theme: {
      type: String,
      required: false,
      default: 'dark'
    },
    collapsible: {
      type: Boolean,
      required: false,
      default: false
    },
    collapsed: {
      type: Boolean,
      required: false,
      default: false
    },
    menus: {
      type: Array,
      required: true
    }
  },
  computed: {
    smenuStyle() {
      let style = { 'padding': '0', 'background': '#222653', 'color': 'white !important' }
      if (this.fixSiderbar) {
        style['height'] = 'calc(100% - 59px)'
        style['overflow'] = 'auto'
        style['overflow-x'] = 'hidden'
      }
      return style
    }
  },
  methods: {
    onSelect(obj) {
      this.$emit('menuSelect', obj)
    },
    onClick(obj) {
      let linkList = ['/backup/reportCenter']
      if (linkList.includes(obj.key)) {
        window.open(obj.key)
        this.$router.go(-1)
      }

    }
  }
}
</script>
<style lang='less' scoped>

/* update_begin author:sunjianlei date:20190509 for: 修改侧边导航栏滚动条的样式 */
.sider {
  @scrollBarSize: 10px;

  ul.ant-menu {

    /* 定义滚动条高宽及背景 高宽分别对应横竖滚动条的尺寸*/

    &::-webkit-scrollbar {
      width: @scrollBarSize;
      height: @scrollBarSize;
      background-color: transparent;
      display: none;
    }

    & .-o-scrollbar {
      display: none;
    }

    /* 兼容IE */
    -ms-overflow-style: none;
    -ms-scroll-chaining: chained;
    -ms-content-zooming: zoom;
    -ms-scroll-rails: none;
    -ms-content-zoom-limit-min: 100%;
    -ms-content-zoom-limit-max: 500%;
    -ms-scroll-snap-type: proximity;
    -ms-scroll-snap-points-x: snapList(100%, 200%, 300%, 400%, 500%);

    /* 定义滚动条轨道 */

    &::-webkit-scrollbar-track {
      background-color: transparent;
    }

    /* 定义滑块 */

    &::-webkit-scrollbar-thumb {
      border-radius: @scrollBarSize;
      background-color: #eee;
      box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.1);

      &:hover {
        background-color: #dddddd;
      }

      &:active {
        background-color: #bbbbbb;
      }
    }
  }

  /** 暗色系滚动条样式 */

  &.dark ul.ant-menu {
    &::-webkit-scrollbar-thumb {
      background-color: #666666;

      &:hover {
        background-color: #808080;
      }

      &:active {
        background-color: #999999;
      }
    }
  }

}

/* update_end author:sunjianlei date:20190509 for: 修改侧边导航栏滚动条的样式 */

</style>

<!-- update_begin author:sunjianlei date:20190530 for: 选中首页的时候不显示背景颜色 -->
<style lang='less'>
.ant-menu.ant-menu-root {
  .ant-menu-item {
    background: transparent !important;
  }

  & > .ant-menu-item:first-child {
    background-color: transparent;

    & > a, & > a:hover {
      color: white;
    }

    &.ant-menu-item-selected {
      & > a, & > a:hover {
        color: white;
      }
    }
  }

  &.ant-menu-dark > .ant-menu-item:first-child {
    & > a, & > a:hover {
      color: rgba(255, 255, 255, 0.65);
    }

    &.ant-menu-item-selected {
      & > a, & > a:hover {
        color: rgba(255, 255, 255, 1);
      }
    }
  }
}

.ant-menu-submenu-title {
  background: #222653 !important;
}

.ant-menu-sub {
  background: #222653 !important;
  color: white !important;
}

.ant-menu-item a {
  color: white !important;
}

.ant-menu-item a:hover {
  color: @primary-color !important;
}

.ant-menu-submenu-selected {
  color: white !important;
}

.ant-menu-item-selected {
  a {
    color: @primary-color !important;
  }
}
</style>
<!-- update_end author:sunjianlei date:20190530 for: 选中首页的时候不显示背景颜色 -->