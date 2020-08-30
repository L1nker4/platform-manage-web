<template>
  <a-card :bordered="false" class="card-area">
    <div :class="advanced ? 'search' : null">

    </div>
    <div>
      <div class="operator">
        <a-button v-hasPermission="['pms:product:add']" type="primary" ghost @click="add">新增</a-button>
        <a-button v-hasPermission="['pms:product:delete']" type="primary" ghost @click="batchDelete">删除</a-button>
<!--        v-hasPermission="['pms:product:export']"-->
        <a-dropdown>
          <a-menu slot="overlay">
            <a-menu-item key="export-data" @click="exportExcel">导出Excel</a-menu-item>
          </a-menu>
          <a-button>
            更多操作 <a-icon type="down" />
          </a-button>
        </a-dropdown>
      </div>
      <!-- 表格区域 -->
      <a-table :columns="columns"
               :dataSource="dataSource"
               :pagination="pagination"
               :row-key="record => record.id"
               :loading="loading"
               :scroll="{ x: 900 }"
               :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
               @change="handleTableChange">
        <template slot="logo" slot-scope="text, record">
          <img v-if="text" :src="text" height="50px" width="50px" />
        </template>

        <template slot="operation" slot-scope="text, record">
          <a-icon v-hasPermission="['pms:product:update']" type="setting" theme="twoTone" twoToneColor="#4a9ff5" @click="edit(record)" title="修改"></a-icon>
          <a-badge v-hasNoPermission="['pms:product:update']" status="warning" text="无权限"></a-badge>
        </template>
      </a-table>
    </div>
    <!-- 新增商品 -->
    <product-add
      @success="handleDeptAddSuccess"
      @close="handleDeptAddClose"
      :productAddVisiable="productAddVisiable">
    </product-add>
    <!-- 修改分类 -->
    <product-edit
      ref="productEdit"
      @success="handleproductEditSuccess"
      @close="handleproductEditClose"
      :productEditVisiable="productEditVisiable">
    </product-edit>
  </a-card>
</template>

<script>

export default {
  name: 'Product',
  components: {},
  data () {
    return {
      advanced: false,
      dataSource: [],
      selectedRowKeys: [],
      queryParams: {},
      sortedInfo: null,
      pagination: {
        defaultPageSize: 10000000,
        hideOnSinglePage: true,
        indentSize: 100
      },
      loading: false,
      productAddVisiable: false,
      productEditVisiable: false
    }
  },
  computed: {
    columns () {
      // let { sortedInfo } = this
      // sortedInfo = sortedInfo || {}
      return [{
        title: '分类名称',
        dataIndex: 'name'
      }, {
        title: '商品数量',
        dataIndex: 'productCount'
      }, {
        title: '商品单位',
        dataIndex: 'productUnit'
      }, {
        title: '显示状态',
        dataIndex: 'showStatus',
        customRender: (text, row, index) => {
          switch (text) {
            case 'NOT_SHOW':
              return '不显示'
            case 'SHOW':
              return '显示'
            default:
              return text
          }
        }
      }, {
        title: '导航栏显示',
        dataIndex: 'navStatus',
        customRender: (text, row, index) => {
          switch (text) {
            case 'NOT_SHOW':
              return '不显示'
            case 'SHOW':
              return '显示'
            default:
              return text
          }
        }
      },{
        title: '图片',
        dataIndex: 'logo',
        scopedSlots: { customRender: 'logo' }
      }, {
        title: '关键字',
        dataIndex: 'keywords'
      }, {
        title: '描述',
        dataIndex: 'description'
      }, {
        title: '操作',
        dataIndex: 'operation',
        scopedSlots: { customRender: 'operation' },
        fixed: 'right',
        width: 120
      }]
    }
  },
  mounted () {
    this.fetch()
  },
  methods: {
    onSelectChange (selectedRowKeys) {
      this.selectedRowKeys = selectedRowKeys
    },
    handleDeptAddClose () {
      this.productAddVisiable = false
    },
    handleDeptAddSuccess () {
      this.productAddVisiable = false
      this.$message.success('新增商品成功')
      this.fetch()
    },
    add () {
      this.productAddVisiable = true
    },
    handleproductEditClose () {
      this.productEditVisiable = false
    },
    handleproductEditSuccess () {
      this.productEditVisiable = false
      this.$message.success('修改商品成功')
      this.fetch()
    },
    edit (record) {
      this.productEditVisiable = true
      this.$refs.productEdit.setFormValues(record)
    },
    handleDateChange (value) {
      if (value) {
        this.queryParams.createTimeFrom = value[0]
        this.queryParams.createTimeTo = value[1]
      }
    },
    batchDelete () {
      if (!this.selectedRowKeys.length) {
        this.$message.warning('请选择需要删除的记录')
        return
      }
      let that = this
      this.$confirm({
        title: '确定删除所选中的记录?',
        content: '当您点击确定按钮后，这些记录将会被彻底删除，如果其包含子记录，也将一并删除！',
        centered: true,
        onOk () {
          that.$delete('product/' + that.selectedRowKeys.join(',')).then(() => {
            that.$message.success('删除成功')
            that.selectedRowKeys = []
            that.fetch()
          })
        },
        onCancel () {
          that.selectedRowKeys = []
        }
      })
    },
    exportExcel () {
      let {sortedInfo} = this
      let sortField, sortOrder
      // 获取当前列的排序和列的过滤规则
      if (sortedInfo) {
        sortField = sortedInfo.field
        sortOrder = sortedInfo.order
      }
      this.$export('product/excel')
    },
    search () {
      let {sortedInfo} = this
      let sortField, sortOrder
      // 获取当前列的排序和列的过滤规则
      if (sortedInfo) {
        sortField = sortedInfo.field
        sortOrder = sortedInfo.order
      }
      this.fetch({
        sortField: sortField,
        sortOrder: sortOrder,
        ...this.queryParams
      })
    },
    reset () {
      // 取消选中
      this.selectedRowKeys = []
      // 重置列排序规则
      this.sortedInfo = null
      // 重置查询参数
      this.queryParams = {}
      // 清空时间选择
      this.$refs.createTime.reset()
      this.fetch()
    },
    handleTableChange (pagination, filters, sorter) {
      this.sortedInfo = sorter
      this.fetch({
        sortField: sorter.field,
        sortOrder: sorter.order,
        ...this.queryParams,
        ...filters
      })
    },
    fetch (params = {}) {
      this.loading = true
      this.$get('/product', {
        ...params
      }).then((r) => {
        let data = r.data.data
        this.loading = false
        this.dataSource = data.rows.children
      })
    }
  }
}
</script>

<style lang="less" scoped>
  @import "../../../../../static/less/Common";
</style>
