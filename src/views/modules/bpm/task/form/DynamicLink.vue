<!--
<template>
  <component :is="component" :formData="formData" v-if="component" />
</template>
<script>
  export default {
    name: 'dynamic-link',
    props: ['formData', 'path'],
    data() {
      return {
        component: null,
      }
    },
    computed: {
      loader() {
        if (!this.path) {
          return null
        }
        return () => import(`@/${this.path}`)
      },
    },
    mounted() {
      let that = this
      that.loader().then(() => {
        that.component = () => that.loader()
        console.log("表单数据",this.formData)
        })
        .catch(() => {
          //that.component = () => import('./FormModule.vue')
        })
    },
  }
</script>
-->

<template>
  <component :is="comp" :formData="formData" v-if="comp"></component>
</template>
<script>
  export default {
    name: 'DynamicLink',
    data () {
      return {
        compName: this.path
      }
    },
    computed: {
      comp: function () {
        return () => import('@/views/' + this.compName + '.vue')
      }
    },
    props: ['path', 'formData']
  }

</script>
