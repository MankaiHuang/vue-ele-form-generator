<template>
  <div class="app-main-header">
    <!-- 顶部按钮 -->
    <el-button @click="isPreview = true" icon="el-icon-view" type="text"
      >预览</el-button
    >
    <el-button @click="isShowData = true" icon="el-icon-upload2" type="text"
      >生成数据</el-button
    >
    <el-button @click="isShowCode = true" icon="el-icon-tickets" type="text"
      >生成代码</el-button
    >
    <el-dialog
      :visible.sync="isPreview"
      append-to-body
      title="预览"
      v-if="isPreview"
      width="90%"
    >
      <ele-form
        :form-desc="formDesc"
        :formData="formData"
        :request-fn="handleRequest"
        @request-success="handleRequestSuccess"
        v-bind="formAttr"
      ></ele-form>
    </el-dialog>
    <!-- 导出数据弹框 -->
    <el-dialog
      :visible.sync="isShowData"
      append-to-body
      title="数据"
      v-if="isShowData"
      width="600px"
    >
      <json-editor :value="codeData"></json-editor>
      <div style="text-align: center;margin-top: 20px">
        <el-button @click="handleCopyData" type="primary">复制数据</el-button>
      </div>
    </el-dialog>

    <!-- 生成代码弹框 -->
    <el-dialog
      :visible.sync="isShowCode"
      append-to-body
      title="代码"
      v-if="isShowCode"
      width="600px"
    >
      <codemirror :value="codeHtml"></codemirror>
      <div style="text-align: center;margin-top: 20px">
        <el-button @click="handleCopyHtml" type="primary">复制代码</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import tpl from '@/tpl'
import { mapState } from 'vuex'
const serialize = require('serialize-javascript')
const copy = require('clipboard-copy')
const cloneDeep = require('lodash.clonedeep')

export default {
  name: 'AppMainHeader',
  computed: {
    ...mapState(['formAttr', 'list']),
    formDesc() {
      const list = cloneDeep(this.list)
      // 将数组转为对象, 并删除无用的属性
      // 例如: [ {filed: 'name', label: '姓名', default: undefined }, [{filed: 'age', label: '年龄', default: 18}] ] => {name: {label: '姓名'}, age: {label: '年龄', default: 18}}
      return list.reduce((acc, formData) => {
        const field = formData['field']
        // 删除字段属性
        delete formData['field']

        // 判断默认值, 如果默认值不存在, 则删除此属性(无需展示)
        if (formData.default === null || formData.default === undefined) {
          delete formData.default
        }

        // 判断布局, layout默认是24, 如果未改变, 则删除此属性(无需展示)
        if (formData.layout === 24) delete formData.layout

        // 删除私有属性
        formData = Object.keys(formData).reduce((res, key) => {
          // _vif, _vshow等
          if (!key.startsWith('_')) res[key] = formData[key]
          return res
        }, {})

        acc[field] = formData
        return acc
      }, {})
    },
    codeHtml() {
      let htmlFormAttr = ''
      const formAttrEntries = Object.entries(this.formAttr)
      // 拼接ele-form属性
      if (formAttrEntries.length) {
        htmlFormAttr = formAttrEntries.reduce((acc, val) => {
          acc.push(`:${val[0]}="${val[1]}"`)
          return acc
        }, [])
        htmlFormAttr = htmlFormAttr.join('\n    ') + '\n    '
      }

      return this.tpl
        .replace('%1', htmlFormAttr)
        .replace('%2', this.serializeObj(this.computedDesc))
    },
    // 数据
    codeData() {
      return Object.assign({}, this.formAttr, {
        formDesc: this.computedDesc
      })
    },

    // 将 desc 去除校检
    computedDesc() {
      const formDesc = cloneDeep(this.formDesc)
      for (let key in formDesc) {
        if (formDesc[key].required) {
          delete formDesc[key].required
        }
      }
      return formDesc
    }
  },
  data() {
    return {
      tpl: tpl,
      formData: {},
      isShowData: false,
      isShowCode: false,
      isPreview: false
    }
  },
  methods: {
    handleCopyData() {
      copy(JSON.stringify(this.codeData))
      this.$message.success('复制成功!')
    },
    handleCopyHtml() {
      copy(this.codeHtml)
      this.$message.success('复制成功!')
    },
    handleRequest(data) {
      console.log(data)
      return Promise.resolve()
    },
    handleRequestSuccess() {
      this.$message.success('发送成功')
    },
    // 序列表对象为字符串
    serializeObj(obj) {
      if (!obj || Object.keys(obj).length === 0) return '{}'
      return serialize(obj, { space: 2 })
        .replace(/"(\w+)":/g, '$1:')
        .replace(/(\s\s)(\S)/g, '      $1$2')
        .replace(/}$/, '      }')
    }
  }
}
</script>

<style>
.app-main-header {
  height: 60px;
  line-height: 60px;
  padding-left: 20px;
  border-bottom: 1px solid #ebeef5;
}
</style>