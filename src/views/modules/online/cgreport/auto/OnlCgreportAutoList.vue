<template>
  <a-card :bordered="false" style="height: 100%">

    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24" v-if="queryInfo && queryInfo.length>0">
          <template v-for="(item,index) in queryInfo">
            <template v-if=" item.hidden==='1' ">
              <a-col v-if="item.view.indexOf('Date')>=0 || item.view.indexOf('date')>=0" :span="item.mode=='single'?6:10" :key=" 'query'+index " v-show="toggleSearchStatus">
                <onl-cgreport-query-form-item :queryParam="queryParam" :item="item" :dictOptions="dictOptions"></onl-cgreport-query-form-item>
              </a-col>
              <a-col v-else :span="item.mode=='single'?6:8" :key=" 'query'+index " v-show="toggleSearchStatus">
                <onl-cgreport-query-form-item :queryParam="queryParam" :item="item" :dictOptions="dictOptions"></onl-cgreport-query-form-item>
              </a-col>
            </template>
            <template v-else>
              <a-col v-if="item.view.indexOf('Date')>=0 || item.view.indexOf('date')>=0" :span="item.mode=='single'?6:10" :key=" 'query'+index ">
                <onl-cgreport-query-form-item :queryParam="queryParam" :item="item" :dictOptions="dictOptions"></onl-cgreport-query-form-item>
              </a-col>
              <a-col v-else :span="item.mode=='single'?6:8" :key=" 'query'+index ">
                <onl-cgreport-query-form-item :queryParam="queryParam" :item="item" :dictOptions="dictOptions"></onl-cgreport-query-form-item>
              </a-col>
            </template>
          </template>

          <a-col :span="6">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchByQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>


    <div class="table-operator" style="margin-bottom: 10px">
      <a-button type="primary" icon="plus" @click="exportExcel">导出</a-button>
    </div>


    <a-table
      ref="table"
      size="middle"
      bordered
      rowKey="id"
      :columns="table.columns"
      :dataSource="table.dataSource"
      :pagination="table.pagination"
      :loading="table.loading"
      :rowSelection="{ selectedRowKeys: table.selectedRowKeys, onChange: handleChangeInTableSelect}"
      @change="handleChangeInTable"
      style="min-height: 300px"
      class="j-table-force-nowrap"
    >

      <!-- 支持链接href跳转 -->
      <template
        v-for="field of fieldHrefSlots"
        :slot="field.slotName"
        slot-scope="text, record"
      >
        <a @click="handleClickFieldHref(field,record)">{{ text }}</a>
      </template>

    </a-table>

    <!-- 跳转Href的动态组件方式 -->
    <a-modal v-bind="hrefComponent.model" v-on="hrefComponent.on">
      <component :is="hrefComponent.is" v-bind="hrefComponent.params"/>
    </a-modal>

  </a-card>
</template>

<script>
  import { HrefJump } from '@/mixins/OnlAutoListMixin'
  import { getAction,downFile } from '@/api/manage'
  import OnlCgreportQueryFormItem from './OnlCgreportQueryFormItem.vue';
  import { filterMultiDictText } from '@/components/dict/JDictSelectUtil'
  import {filterObj} from '@/utils/util';

  export default {
    name: 'OnlCgreportAutoList',
    mixins: [HrefJump],
    components: {
      OnlCgreportQueryFormItem
    },
    data() {
      return {
        // 查询参数
        queryInfo: [],
        // 查询参数，多个页面的查询参数用 code 作为键来区分
        queryParamsMap: {},
        selfParam:{
        },
        sorter: {
          column: '',
          order: 'desc',
        },
        dictOptions: {},
        toggleSearchStatus: false, // 高级搜索 展开/关闭
        reportCode: '',
        description: '在线报表功能测试页面',
        url: {
          getColumnsAndData: '/online/cgreport/api/getColumnsAndData/',
          getQueryInfo: '/online/cgreport/api/getQueryInfo/',
          getParamsInfo:'/online/cgreport/api/getParamsInfo/'
        },
        table: {
          loading: true,
          // 表头
          columns: [],
          //数据集
          dataSource: [],
          // 选择器
          selectedRowKeys: [],
          selectionRows: [],
          // scroll: { x: false },
          // 分页参数
          pagination: {
            current: 1,
            pageSize: 10,
            pageSizeOptions: ['10', '20', '30'],
            showTotal: (total, range) => {
              return range[0] + '-' + range[1] + ' 共' + total + '条'
            },
            showQuickJumper: true,
            showSizeChanger: true,
            total: 0
          }
        },
        cgreportHeadName:""
      }
    },
    mounted() {
      this.initParamsInfo()
      this.initQueryInfo();
    },
    watch: {
      '$route'() {
        // 刷新参数放到这里去触发，就可以刷新相同界面了
        this.initParamsInfo()
        this.initQueryInfo();
      }
    },
    computed: {
      queryParam: {
        get() {
          return this.queryParamsMap[this.reportCode]
        },
        set(newVal) {
          this.$set(this.queryParamsMap, this.reportCode, newVal)
        }
      }
    },
    methods: {
      initParamsInfo(){
        if(!this.$route.params.code){
          return false
        }
        //获取报表ID
        this.reportCode = this.$route.params.code;
        if (!this.queryParam) {
          this.queryParam = {}
        }

        this.selfParam={}
        getAction(`${this.url.getParamsInfo}${this.reportCode}`).then((res) => {
          if (res.success) {
            if(res.result && res.result.length>0){
              for(let i of res.result){
                this.selfParam['self_'+i.paramName]=(!this.$route.query[i.paramName])?"":this.$route.query[i.paramName]
              }
            }
          } else {
            this.$message.warning(res.message)
          }
          this.loadData()
        })
      },
      initQueryInfo() {
        if(!this.reportCode){
          return false
        }
        getAction(`${this.url.getQueryInfo}${this.reportCode}`).then((res) => {
          console.log("获取查询条件", res);
          if (res.success) {
            this.queryInfo = res.result
          } else {
            this.$message.warning(res.message)
          }
        })
      },
      loadData(arg) {
        if(!this.reportCode){
          return false
        }
        if (arg == 1) {
          this.table.pagination.current = 1
        }
        let params = this.getQueryParams();//查询条件
        console.log(params)

        //获取报表ID
        console.log(' 动态报表 reportCode ： ' + this.reportCode);
        this.table.loading = true

        getAction(`${this.url.getColumnsAndData}${this.reportCode}`, params).then(res => {
          if (res.success) {
            let { data, columns, cgreportHeadName, dictOptions, fieldHrefSlots } = res.result
            // href 跳转
            const fieldHrefSlotKeysMap = {}
            fieldHrefSlots.forEach(item => fieldHrefSlotKeysMap[item.slotName] = item)
            // let columnWidth = 230
            this.dictOptions = dictOptions
            columns.forEach(column => {
              // column.width = columnWidth
              // 处理列中的 href 跳转和 dict 字典，使两者可以兼容存在
              this.handleColumnHrefAndDict(column, fieldHrefSlotKeysMap)
            })
            // this.table.scroll.x = columns.length * columnWidth
            this.table.columns = [...columns]
            this.cgreportHeadName = cgreportHeadName
            this.fieldHrefSlots = fieldHrefSlots
            if (data) {
              this.table.pagination.total = Number(data.total)
              this.table.dataSource = data.records
            } else {
              this.table.pagination.total = 0
              this.table.dataSource = []
            }

          }else{
            this.$message.warn('查询失败：'+res.message)
          }

        }).catch((e) => {
          console.error(e)
          this.$message.error('查询失败')
        }).finally(() => {
          this.table.loading = false
        })
      },
      getQueryParams() {
        let param = Object.assign({}, this.queryParam, this.sorter,this.selfParam);
        param.pageNo = this.table.pagination.current;
        param.pageSize = this.table.pagination.pageSize;
        return filterObj(param);
      },
      searchByQuery() {
        this.loadData(1);
      },
      searchReset() {
        this.queryParam = {}
        this.loadData(1);
      },
      handleToggleSearch() {
        this.toggleSearchStatus = !this.toggleSearchStatus;
      },
      exportExcel() {
        let fileName = this.cgreportHeadName
        let selfParam = {}
        for (let queryName in this.$route.query) {
          if (this.$route.query.hasOwnProperty(queryName)) {
            let value = this.$route.query[queryName]
            selfParam['self_' + queryName] = value || ''
          }
        }
        downFile(`/online/cgreport/api/exportXls/${this.reportCode}`, Object.assign(selfParam, this.queryParam)).then((data) => {
          if (!data) {
            this.$message.warning("文件下载失败")
            return
          }
          if (typeof window.navigator.msSaveBlob !== 'undefined') {
            window.navigator.msSaveBlob(new Blob([data]), fileName+'.xls')
          }else{
            let url = window.URL.createObjectURL(new Blob([data]))
            let link = document.createElement('a')
            link.style.display = 'none'
            link.href = url
            link.setAttribute('download', fileName+'.xls')
            document.body.appendChild(link)
            link.click()
            document.body.removeChild(link); //下载完成移除元素
            window.URL.revokeObjectURL(url); //释放掉blob对象
          }
        })
      },
      handleChangeInTableSelect(selectedRowKeys, selectionRows) {
        this.table.selectedRowKeys = selectedRowKeys
        this.table.selectionRows = selectionRows
      },
      handleChangeInTable(pagination, filters, sorter) {
        //分页、排序、筛选变化时触发
        if (Object.keys(sorter).length > 0) {
          this.sorter.column = sorter.field
          this.sorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
        } else {
          this.sorter.column = null
          this.sorter.order = null
        }
        this.table.pagination = pagination
        this.loadData()
      }

    }
  }
</script>
<style scoped>
  .div {
    display: flex;
    align-items: center;
    height: 500px
  }
</style>
<style lang="less">
  @import '~@/assets/less/TableExpand.less';
</style>