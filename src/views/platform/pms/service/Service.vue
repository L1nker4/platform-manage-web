<template>
  <a-card :bordered="false" class="card-area">
    <div>
      <div class="operator">
        <a-button v-hasPermission="['pms:service:add']" type="primary" ghost @click="add">新增</a-button>
        <a-button
          v-hasPermission="['pms:service:delete']"
          type="primary"
          ghost
          @click="batchDelete"
        >删除</a-button>
        <a-dropdown>
          <a-menu slot="overlay" v-hasPermission="['pms:service:export']">
            <a-menu-item key="export-data" @click="exportExcel">导出Excel</a-menu-item>
          </a-menu>
          <a-button>
            更多操作
            <a-icon type="down" />
          </a-button>
        </a-dropdown>
      </div>
      <!-- 表格区域 -->
      <a-table
        :columns="columns"
        :dataSource="dataSource"
        :pagination="pagination"
        :row-key="record => record.id"
        :loading="loading"
        :scroll="{ x: 900 }"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange"
      >
        <template slot="logo" slot-scope="text, record">
          <img v-if="text" :src="text" height="50px" width="50px" />
        </template>

        <template slot="operation" slot-scope="text, record">
          <a-icon
            v-hasPermission="['pms:service:update']"
            type="setting"
            theme="twoTone"
            twoToneColor="#4a9ff5"
            @click="edit(record)"
            title="修改"
          ></a-icon>
          <a-badge v-hasNoPermission="['pms:service:update']" status="warning" text="无权限"></a-badge>
        </template>
      </a-table>
    </div>
    <a-drawer
      title="修改商品服务"
      :maskClosable="false"
      width="650"
      placement="right"
      :closable="false"
      @close="onClose"
      :visible="serviceEditVisiable"
      style="height: calc(100% - 55px);overflow: auto;padding-bottom: 53px;"
    >
      <a-form-model ref="checkForm" :model="form">
        <a-form-model-item label="服务名称" prop="name">
          <a-input v-model="form.name" />
        </a-form-model-item>
        <a-form-model-item label="Logo" prop="logo">
          <img v-if="form.logo" :src="form.logo" height="50px" width="50px" />
          <a-upload
            :action="uploadUrl"
            class="upload-list-inline"
            @change="handleImageChange"
            :headers="headers"
          >
            <a-button>
              <a-icon type="upload" />上传logo
            </a-button>
          </a-upload>
        </a-form-model-item>
        <a-form-model-item label="描述" prop="description">
          <a-textarea v-model="form.description" :auto-size="{ minRows: 3, maxRows: 5 }" />
        </a-form-model-item>
      </a-form-model>
      <div class="drawer-bootom-button">
        <a-popconfirm title="确定放弃编辑？" @confirm="onClose" okText="确定" cancelText="取消">
          <a-button style="margin-right: .8rem">取消</a-button>
        </a-popconfirm>
        <a-button @click="handleSubmit" type="primary" :loading="loading">提交</a-button>
      </div>
    </a-drawer>

    <a-drawer
      title="添加商品服务"
      :maskClosable="false"
      width="650"
      placement="right"
      :closable="false"
      @close="onCloseAdd"
      :visible="serviceAddVisiable"
      style="height: calc(100% - 55px);overflow: auto;padding-bottom: 53px;"
    >
      <a-form-model ref="checkForm" :model="addForm">
        <a-form-model-item label="服务名称" prop="name">
          <a-input v-model="addForm.name" />
        </a-form-model-item>
        <a-form-model-item label="Logo" prop="logo">
          <img v-if="addForm.logo" :src="addForm.logo" height="50px" width="50px" />
          <a-upload
            :action="uploadUrl"
            class="upload-list-inline"
            @change="handleImageChange"
            :headers="headers"
          >
            <a-button>
              <a-icon type="upload" />上传logo
            </a-button>
          </a-upload>
        </a-form-model-item>
        <a-form-model-item label="描述" prop="description">
          <a-textarea v-model="addForm.description" :auto-size="{ minRows: 3, maxRows: 5 }" />
        </a-form-model-item>
      </a-form-model>
      <div class="drawer-bootom-button">
        <a-popconfirm title="确定放弃编辑？" @confirm="onCloseAdd" okText="确定" cancelText="取消">
          <a-button style="margin-right: .8rem">取消</a-button>
        </a-popconfirm>
        <a-button @click="addSubmit" type="primary" :loading="loading">提交</a-button>
      </div>
    </a-drawer>
  </a-card>
</template>

<script>
import store from "../../../../store";

export default {
  name: "Service",
  components: {},
  data() {
    return {
      dataSource: [],
      pagination: {
        defaultPageSize: 10000000,
        hideOnSinglePage: true,
        indentSize: 100,
      },
      form: {
        id: "",
        name: "",
        logo: "",
        description: "",
      },
      addForm: {
        name: "",
        logo: "",
        description: "",
      },
      uploadUrl: "http://localhost:9009/upload",
      headers: {
        Authentication: store.state.account.token,
      },
      selectedRowKeys: [],
      loading: false,
      queryParams: {},
      sortedInfo: null,
      serviceEditVisiable: false,
      serviceAddVisiable: false,
    };
  },
  computed: {
    columns() {
      // let { sortedInfo } = this
      // sortedInfo = sortedInfo || {}
      return [
        {
          title: "服务名称",
          dataIndex: "name",
        },
        {
          title: "Logo",
          dataIndex: "logo",
          scopedSlots: { customRender: "logo" },
        },
        {
          title: "描述",
          dataIndex: "description",
        },
        {
          title: "操作",
          dataIndex: "operation",
          scopedSlots: { customRender: "operation" },
          fixed: "right",
          width: 120,
        },
      ];
    },
  },
  mounted() {
    this.fetch();
  },
  methods: {
    handleSubmit() {
      this.$put("service", this.form)
        .then(() => {
          this.reset();
          this.$message.info('修改成功')
        })
        .catch(() => {
          this.loading = false;
        });
    },
    addSubmit() {
      console.log(this.addForm);
      this.$post('service', this.addForm).then(() => {
            this.addForm = {}
            this.serviceAddVisiable = false
            this.fetch()
            this.$message.info('添加成功')
          }).catch(() => {
            this.loading = false
          })
    },
    reset() {
      this.serviceEditVisiable = false;
      this.form = {};
    },
    handleImageChange(value) {
      if (value.file.status === "done") {
        this.form.logo = value.file.response.data;
        this.addForm.logo = value.file.response.data;
      }
      // this.category.logo = value.file.response.data
    },
    add() {
      this.serviceAddVisiable = true;
    },
    onClose() {
      this.reset();
      this.$emit("close");
    },
    onCloseAdd() {
      this.addForm = {};
      this.serviceAddVisiable = false;
      this.reset();
      this.$emit("close");
    },
    edit(record) {
      this.serviceEditVisiable = true;
      this.form = record;
    },
    exportExcel() {
      let { sortedInfo } = this;
      let sortField, sortOrder;
      // 获取当前列的排序和列的过滤规则
      if (sortedInfo) {
        sortField = sortedInfo.field;
        sortOrder = sortedInfo.order;
      }
      this.$export("service/excel");
    },
    batchDelete() {
      if (!this.selectedRowKeys.length) {
        this.$message.warning("请选择需要删除的记录");
        return;
      }
      let that = this;
      this.$confirm({
        title: "确定删除所选中的记录?",
        content:
          "当您点击确定按钮后，这些记录将会被彻底删除，如果其包含子记录，也将一并删除！",
        centered: true,
        onOk() {
          let id = that.selectedRowKeys.join(",")
          console.log(id);
          that.$delete("service/" + id).then(() => {
            that.$message.success("删除成功");
            that.selectedRowKeys = [];
            that.fetch();
          });
        },
        onCancel() {
          that.selectedRowKeys = [];
        },
      });
    },
    handleTableChange(pagination, filters, sorter) {
      this.sortedInfo = sorter;
      this.fetch({
        sortField: sorter.field,
        sortOrder: sorter.order,
        ...this.queryParams,
        ...filters,
      });
    },
    onSelectChange(selectedRowKeys) {
      this.selectedRowKeys = selectedRowKeys;
    },
    fetch(params = {}) {
      this.loading = true;
      this.$get("/service", {
        ...params,
      }).then((r) => {
        console.log(r);
        let data = r.data.data;
        this.loading = false;
        this.dataSource = data.records;
      });
    },
  },
};
</script>

<style lang="less" scoped>
@import "../../../../../static/less/Common";
</style>
