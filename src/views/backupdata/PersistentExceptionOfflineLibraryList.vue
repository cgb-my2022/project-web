<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="备份域">
              <a-input placeholder="请输入备份域" v-model="queryParam.domain"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="库">
              <a-input placeholder="请输入库" v-model="queryParam.library"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="型号">
                <a-input placeholder="请输入型号" v-model="queryParam.model"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="版本">
                <a-input placeholder="请输入版本" v-model="queryParam.version"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="10" :lg="11" :md="12" :sm="24">
              <a-form-item label="安装时间">
                <j-date :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" placeholder="请选择开始时间" class="query-group-cust" v-model="queryParam.installTime_begin"></j-date>
                <span class="query-group-split-cust"></span>
                <j-date :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" placeholder="请选择结束时间" class="query-group-cust" v-model="queryParam.installTime_end"></j-date>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="默认暂存池">
                <a-input placeholder="请输入默认暂存池" v-model="queryParam.defaultCache"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="默认介质类型">
                <a-input placeholder="请输入默认介质类型" v-model="queryParam.defaultMediumType"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="超时(装载/卸载)">
                <a-input placeholder="请输入超时(装载/卸载)" v-model="queryParam.overTime"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="驱动器数/介质数">
                <a-input placeholder="请输入驱动器数/介质数" v-model="queryParam.mediumNumber"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="槽数(邮件槽/空/总数)">
                <a-input placeholder="请输入槽数(邮件槽/空/总数)" v-model="queryParam.grooveNumber"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="错误(软/硬)">
                <a-input placeholder="请输入错误(软/硬)" v-model="queryParam.error"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="状态">
                <a-input placeholder="请输入状态" v-model="queryParam.state"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="关联的用户组">
                <a-input placeholder="请输入关联的用户组" v-model="queryParam.relationGroup"></a-input>
              </a-form-item>
            </a-col>
          </template>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
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
    <!-- 查询区域-END -->
    
    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('持续异常库脱机')">导出</a-button>
      <a-upload
        name="file"
        :showUploadList="false"
        :multiple="false"
        :headers="tokenHeader"
        :action="importExcelUrl"
        @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        class="j-table-force-nowrap"
        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="uploadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>

    <persistentExceptionOfflineLibrary-modal ref="modalForm" @ok="modalFormOk"></persistentExceptionOfflineLibrary-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import PersistentExceptionOfflineLibraryModal from './modules/PersistentExceptionOfflineLibraryModal'
  import JDate from '@/components/jeecg/JDate.vue'

  export default {
    name: "PersistentExceptionOfflineLibraryList",
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      JDate,
      PersistentExceptionOfflineLibraryModal
    },
    data () {
      return {
        description: '持续异常库脱机管理页面',
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
          {
            title:'备份域',
            align:"center",
            dataIndex: 'domain'
          },
          {
            title:'库',
            align:"center",
            dataIndex: 'library'
          },
          {
            title:'型号',
            align:"center",
            dataIndex: 'model'
          },
          {
            title:'版本',
            align:"center",
            dataIndex: 'version'
          },
          {
            title:'安装时间',
            align:"center",
            dataIndex: 'installTime'
          },
          {
            title:'默认暂存池',
            align:"center",
            dataIndex: 'defaultCache'
          },
          {
            title:'默认介质类型',
            align:"center",
            dataIndex: 'defaultMediumType'
          },
          {
            title:'超时(装载/卸载)',
            align:"center",
            dataIndex: 'overTime'
          },
          {
            title:'驱动器数/介质数',
            align:"center",
            dataIndex: 'mediumNumber'
          },
          {
            title:'槽数(邮件槽/空/总数)',
            align:"center",
            dataIndex: 'grooveNumber'
          },
          {
            title:'错误(软/硬)',
            align:"center",
            dataIndex: 'error'
          },
          {
            title:'状态',
            align:"center",
            dataIndex: 'state'
          },
          {
            title:'关联的用户组',
            align:"center",
            dataIndex: 'relationGroup'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            // fixed:"right",
            width:147,
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          list: "/persistentexceptionofflinelibrary/persistentExceptionOfflineLibrary/list",
          delete: "/persistentexceptionofflinelibrary/persistentExceptionOfflineLibrary/delete",
          deleteBatch: "/persistentexceptionofflinelibrary/persistentExceptionOfflineLibrary/deleteBatch",
          exportXlsUrl: "/persistentexceptionofflinelibrary/persistentExceptionOfflineLibrary/exportXls",
          importExcelUrl: "persistentexceptionofflinelibrary/persistentExceptionOfflineLibrary/importExcel",
        },
        dictOptions:{},
      }
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      },
    },
    methods: {
      initDictConfig(){
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>