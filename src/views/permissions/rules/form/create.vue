<template>
  <div>
    <a-modal
      :title="title"
      :width="640"
      :visible="visible"
      :confirmLoading="confirmLoading"
      @ok="handleSubmit"
      @cancel="handleCancel"
    >
      <a-spin :spinning="confirmLoading">
        <a-form :form="form">
          <a-form-item
            label="菜单名称"
            type="text"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
          >
            <a-input allowClear v-decorator="['permission_name', {rules: [{required: true, min: 2, message: '请输入至少3个字符！'}]}]" />
          </a-form-item>
          <a-form-item
            label="菜单图标"
            type="text"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
          >
            <a-input v-decorator="['icon']" @click="selectIcon" />
          </a-form-item>
          <a-form-item
            label="菜单路由"
            type="text"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
          >
            <a-input allowClear v-decorator="['route', {rules: [{required: true, min: 2, message: '请输入至少3个字符！'}]}]" />
          </a-form-item>
          <a-form-item
            label="模块"
            type="text"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            v-if="module_show"
          >
            <a-input allowClear v-decorator="['module', {rules: [{required: true, min: 2, message: '请输入至少3个字符！'}]}]" />
          </a-form-item>
          <a-form-item
            label="权限标识"
            type="text"
            :labelCol="labelCol"
            :wrapperCol="wrapperCol"
            :filterOption="filterOption"
          >
            <a-input allowClear v-decorator="['permission_mark',{rules: [{required: true, min: 2, message: '请输入至少3个字符！'}]}]" />
          </a-form-item>
          <a-form-item
            :label-col="labelCol"
            :wrapper-col="wrapperCol"
            label="请求方法"
          >
            <a-select v-decorator="['method',{initialValue:methodValue},{rules: [{required: true}]}]">
              <a-select-option value="get">
                get
              </a-select-option>
              <a-select-option value="post">
                post
              </a-select-option>
              <a-select-option value="put">
                put
              </a-select-option>
              <a-select-option value="delete">
                delete
              </a-select-option>
            </a-select>
          </a-form-item>
          <a-form-item
            :label-col="labelCol"
            :wrapper-col="wrapperCol"
            label="类型"
          >
            <a-radio-group buttonStyle="solid" v-decorator="['type',{initialValue: typeValue},{rules: [{required: true}]}]">
              <a-radio-button :value="1">菜单</a-radio-button>
              <a-radio-button :value="2">按钮</a-radio-button>
            </a-radio-group>
          </a-form-item>
          <a-form-item
            :label-col="labelCol"
            :wrapper-col="wrapperCol"
            label="排序"
          >
            <a-input-number :min="1" v-decorator="['sort', {initialValue: sort}]" />
          </a-form-item>
        </a-form>
      </a-spin>
    </a-modal>
    <a-modal title="选择图标" v-model="iconVisible">
      <icon-selector zIndex="2000" :width="1200" v-model="currentSelectedIcon" @change="handleIconChange"/>
    </a-modal>
  </div>
</template>

<script>
import { store, update } from '@/api/permission'
import pick from 'lodash.pick'
import IconSelector from '@/components/IconSelector'

export default {
  components: {
    IconSelector
  },
  data () {
    return {
      labelCol: {
        xs: { span: 24 },
        sm: { span: 7 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 13 }
      },
      visible: false,
      title: '新建菜单',
      confirmLoading: false,
      id: null,
      parent_id: 0,
      methodValue: 'GET',
      typeValue: 2,
      form: this.$form.createForm(this),
      sort: 1,
      module_show: true,
      currentSelectedIcon: '',
      iconVisible: false
    }
  },
  methods: {
    selectIcon () {
      this.iconVisible = true
    },
    handleIconChange (icon) {
      this.$message.info(<span>选中图标 <code>{icon}</code></span>)
      this.form.setFieldsValue({
        icon: icon
      })
      this.iconVisible = false
    },
    add () {
      this.visible = true
      this.title = '新增菜单'
    },
    edit (record) {
      this.visible = true
      this.title = '编辑菜单'
      const { form: { setFieldsValue } } = this
      this.id = record.id
      this.$nextTick(() => {
        setFieldsValue(pick(record, ['permission_name', 'route', 'module', 'permission_mark', 'method', 'type', 'sort', 'icon']))
      })
      this.methodValue = record.method
      this.typeValue = record.type
      this.sort = record.sort
      if (record.parent_id > 0) {
        this.module_show = false
      }
    },
    addSon (record) {
      this.visible = true
      this.title = '新增子菜单 (' + record.permission_name + ')'
      this.parent_id = record.id
      this.module_show = false
    },
    handleSubmit () {
      const { form: { validateFields } } = this
      if (this.id) {
        validateFields((errors, values) => {
          if (!errors) {
            this.confirmLoading = true
            update(this.id, values).then((res) => {
              this.refresh(res.message)
            }).catch(err => this.failed(err))
          }
        })
      } else {
        validateFields((errors, values) => {
          if (!errors) {
            this.confirmLoading = true
            if (this.parent_id > 0) {
              values['parent_id'] = this.parent_id
            }
            store(values).then((res) => {
              this.refresh(res.message)
            }).catch(err => this.failed(err))
          }
        })
      }
    },
    failed (errors) {
      this.$notification['error']({
        message: errors.message,
        duration: 4
      })
      this.handleCancel()
    },
    handleCancel () {
      this.visible = false
      this.id = null
      this.confirmLoading = false
      this.parent_id = 0
      this.methodValue = 'GET'
      this.typeValue = 2
      this.sort = 1
      this.form.resetFields()
    },
    filterOption (input, option) {
      return (
        option.componentOptions.children[0].text.toLowerCase().indexOf(input.toLowerCase()) >= 0
      )
    },
    refresh (message) {
      this.$notification['success']({
        message: message,
        duration: 4
      })
      this.handleCancel()
      this.$parent.$parent.handleOk()
    }
  }
}
</script>
