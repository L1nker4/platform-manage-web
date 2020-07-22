<template>
  <a-drawer
    title="新增商品分类"
    :maskClosable="false"
    width=650
    placement="right"
    :closable="false"
    @close="onClose"
    :visible="categoryAddVisiable"
    style="height: calc(100% - 55px);overflow: auto;padding-bottom: 53px;">
    <a-form :form="form">
      <a-form-item label='分类名称' v-bind="formItemLayout">
        <a-input v-decorator="['name',
                   {rules: [
                    { required: true, message: '分类名称不能为空'},
                    { max: 20, message: '长度不能超过20个字符'}
                  ]}]"/>
      </a-form-item>
      <a-form-item label='商品数量' v-bind="formItemLayout">
        <a-input v-decorator="['productCount',
                   {rules: [
                    { required: true, message: '商品数量不能为空'},
                    { max: 10, message: '长度不能超过10个字符'}
                  ]}]"/>
      </a-form-item>
      <a-form-item label='商品单位' v-bind="formItemLayout">
        <a-input v-decorator="['productUnit',
                   {rules: [
                    { required: true, message: '商品单位不能为空'},
                    { max: 10, message: '长度不能超过10个字符'}
                  ]}]"/>
      </a-form-item>
      <a-form-item label='导航显示' v-bind="formItemLayout">
        <a-select v-decorator="['navStatus',
                   {rules: [
                    { required: true, message: '导航栏显示不能为空'}
                  ]}]"
                   @change="handleNavChange">
          <a-select-option value="0">
            不显示
          </a-select-option>
          <a-select-option value="1">
            显示
          </a-select-option>
        </a-select>
      </a-form-item>
      <a-form-item label='显示状态' v-bind="formItemLayout">
        <a-select v-decorator="['showStatus',
                   {rules: [
                    { required: true, message: '显示状态显示不能为空'}
                  ]}]"
                  @change="handleShowChange">
          <a-select-option value="0">
            不显示
          </a-select-option>
          <a-select-option value="1">
            显示
          </a-select-option>
        </a-select>
      </a-form-item>
      <a-form-item label='排序' v-bind="formItemLayout">
        <a-input-number v-decorator="['sort',
                   {rules: [
                    { required: true, message: '排序不能为空'}
                  ]}]" style="width: 100%"/>
      </a-form-item>
      <a-form-item label='关键字' v-bind="formItemLayout">
        <a-input v-decorator="['keywords',
                   {rules: [
                    { required: true, message: '关键字不能为空'},
                    { max: 10, message: '长度不能超过10个字符'}
                  ]}]"/>
      </a-form-item>
      <a-form-item label='分类logo' v-bind="formItemLayout">
        <a-upload
          :action="uploadUrl"
          class="upload-list-inline"
          @change="handleImageChange"
          :headers="headers">
          <a-button> <a-icon type="upload" /> 上传分类logo </a-button>
        </a-upload>
      </a-form-item>
      <a-form-item label='描述' v-bind="formItemLayout">
        <a-input v-decorator="['description',
                   {rules: [
                    { required: true, message: '描述不能为空'},
                    { max: 10, message: '长度不能超过10个字符'}
                  ]}]"/>
      </a-form-item>
      <a-form-item label='分类'
                   style="margin-bottom: 2rem"
                   v-bind="formItemLayout">
        <a-tree
          :key="categoryTreeKeys"
          :checkable="true"
          :checkStrictly="true"
          @check="handleCheck"
          @expand="handleExpand"
          :expandedKeys="expandedKeys"
          :treeData="categoryTreeData">
        </a-tree>
      </a-form-item>
    </a-form>
    <div class="drawer-bootom-button">
      <a-popconfirm title="确定放弃编辑？" @confirm="onClose" okText="确定" cancelText="取消">
        <a-button style="margin-right: .8rem">取消</a-button>
      </a-popconfirm>
      <a-button @click="handleSubmit" type="primary" :loading="loading">提交</a-button>
    </div>
  </a-drawer>
</template>
<script>
import StyleItem from "../../../../components/setting/StyleItem";
import store from "../../../../store"

const formItemLayout = {
  labelCol: { span: 4 },
  wrapperCol: { span: 18 }
}
export default {
  name: 'CategoryAdd',
  components: {StyleItem},
  props: {
    categoryAddVisiable: {
      default: false
    }
  },
  data () {
    return {
      loading: false,
      formItemLayout,
      fileList: [],
      headers: {
        Authentication: store.state.account.token,
      },
      // uploadUrl: base_url + '/upload/upload',
      uploadUrl: 'http://localhost:9009/upload',
      form: this.$form.createForm(this),
      category: {},
      checkedKeys: [],
      expandedKeys: [],
      categoryTreeData: [],
      categoryTreeKeys: +new Date()
    }
  },
  methods: {
    reset () {
      this.loading = false
      this.categoryTreeKeys = +new Date()
      this.expandedKeys = this.checkedKeys = []
      this.category = {}
      this.form.resetFields()
    },
    onClose () {
      this.reset()
      this.$emit('close')
    },
    handleCheck (checkedKeys) {
      this.checkedKeys = checkedKeys
    },
    handleExpand (expandedKeys) {
      this.expandedKeys = expandedKeys
    },
    handleSubmit () {
      let checkedArr = Object.is(this.checkedKeys.checked, undefined) ? this.checkedKeys : this.checkedKeys.checked
      if (checkedArr.length > 1) {
        this.$message.error('最多只能选择一个上级分类，请修改')
        return
      }
      this.form.validateFields((err, values) => {
        if (!err) {
          this.setCategoryFields()
          this.loading = true
          if (checkedArr.length) {
            this.category.parentId = checkedArr[0]
          } else {
            this.category.parentId = 0
          }
          console.log(this.category)
          this.$post('category', this.category).then(() => {
            this.reset()
            this.$emit('success')
          }).catch(() => {
            this.loading = false
          })
        }
      })
    },
    handleImageChange(value) {
      console.log(value.file)
      if (value.file.status === 'done') {
        this.category.logo = value.file.response.data
      }
      // this.category.logo = value.file.response.data
    },
    handleNavChange (value) {
      this.form.setFieldsValue({
        navStatus: value
      })
      // this.form.validateFields((err, values) => {
      //   if (!err) {
      //     console.log('Received values of form: ', values);
      //   }
      // })
    },
    handleShowChange(value) {
      this.form.setFieldsValue({
        showStatus: value
      })
    },
    setCategoryFields () {
      let values = this.form.getFieldsValue()
      if (typeof values !== 'undefined') {
        Object.keys(values).forEach(_key => { this.category[_key] = values[_key] })
      }
    }
  },
  watch: {
    categoryAddVisiable () {
      if (this.categoryAddVisiable) {
        this.$get('category').then((r) => {
          this.categoryTreeData = r.data.data.rows.children
        })
      }
    }
  }
}
</script>

<style scoped>
  .upload-list-inline >>> .ant-upload-list-item {
    float: left;
    width: 200px;
    margin-right: 8px;
  }
  .upload-list-inline >>> .ant-upload-animate-enter {
    animation-name: uploadAnimateInlineIn;
  }
  .upload-list-inline >>> .ant-upload-animate-leave {
    animation-name: uploadAnimateInlineOut;
  }
</style>
