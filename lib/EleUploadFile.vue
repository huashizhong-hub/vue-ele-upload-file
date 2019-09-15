<template>
  <div>
    <el-upload
      :accept="accept"
      :action="action"
      :before-upload="handleBeforeUpload"
      :data="data"
      :drag="false"
      :file-list="list"
      :headers="headers"
      :limit="limit"
      :multiple="multiple"
      :name="name"
      :on-error="handleUploadError"
      :on-exceed="handleExceed"
      :on-success="handleUploadSuccess"
      :show-file-list="false"
      :withCredentials="withCredentials"
      ref="upload"
      v-if="!disabled"
    >
      <!-- 上传按钮 -->
      <el-button
        size="medium"
        type="primary"
      >{{btnText}}</el-button>
      <!-- 上传提示 -->
      <div
        class="el-upload__tip"
        slot="tip"
        v-if="showTip"
      >
        请上传
        <template v-if="fileSize">
          大小不超过
          <b style="color: #F56C6C">{{fileSize}}MB</b>
        </template>
        <template v-if="fileType">
          格式为
          <b style="color: #F56C6C">{{fileType.join('/')}}</b>
        </template>
        的文件
      </div>
    </el-upload>

    <!-- 列表 -->
    <ele-upload-list
      :disabled="disabled"
      :files="list"
      :isCanDelete="isCanDelete"
      :isCanDownload="isCanDownload"
      :isShowSize="isShowSize"
      @remove="handleRemove"
    />
  </div>
</template>

<script>
import EleUploadList from './EleUploadList'
export default {
  name: 'EleUploadFile',
  components: {
    EleUploadList
  },
  props: {
    // 值
    value: [String, Object, Array],
    // 必选参数，上传的地址
    // 同 element-ui upload 组件
    action: {
      type: String,
      required: true
    },
    // 大小限制(MB)
    fileSize: Number,
    // 响应处理函数
    responseFn: Function,
    // 文件类型, 例如['png', 'jpg', 'jpeg']
    fileType: Array,
    // 提示
    placeholder: String,
    // 是否禁用
    disabled: Boolean,
    // 是否显示文件大小
    isShowSize: {
      type: Boolean,
      default: true
    },
    // 是否显示下载
    isCanDownload: {
      type: Boolean,
      default: true
    },
    // 是否可删除
    isCanDelete: {
      type: Boolean,
      default: true
    },
    // 是否可上传相同文件
    isCanUploadSame: {
      type: Boolean,
      default: true
    },
    // 是否显示提示
    isShowTip: {
      type: Boolean,
      default: true
    },
    // 设置上传的请求头部
    // 同 element-ui upload 组件
    headers: Object,
    // 是否支持多选文件
    // 同 element-ui upload 组件
    multiple: {
      type: Boolean,
      default: true
    },
    // 上传时附带的额外参数
    // 同 element-ui upload 组件
    data: Object,
    // 上传的文件字段名
    // 同 element-ui upload 组件
    name: {
      type: String,
      default: 'file'
    },
    // 支持发送 cookie 凭证信息
    // 同 element-ui upload 组件
    withCredentials: {
      type: Boolean,
      default: false
    },
    // 接受上传的文件类型
    // 同 element-ui upload 组件
    accept: String,
    // 最大允许上传个数
    // 同 element-ui upload 组件
    limit: Number
  },
  computed: {
    // 按钮文本
    btnText () {
      if (this.placeholder) {
        return this.placeholder
      } else {
        if (this.multiple) {
          return '上传文件'
        } else {
          return '上传单个文件'
        }
      }
    },
    // 是否显示提示
    showTip() {
      return this.isShowTip && (this.fileType || this.fileSize)
    },
    // 列表
    list () {
      if (this.value) {
        // 首先将值转为数组
        const list = Array.isArray(this.value) ? this.value : [this.value]
        // 然后将数组转为对象数组
        return list.map((item) => typeof item === 'string' ? { name: item, url: item } : item)
      } else {
        return []
      }
    }
  },
  methods: {
    // 上传前校检格式和大小
    handleBeforeUpload (file) {
      // 校检文件类型
      if (this.fileType) {
        let fileExtension = ''
        if (file.name.lastIndexOf('.') > -1) {
          fileExtension = file.name.slice(file.name.lastIndexOf('.') + 1)
        }
        const isTypeOk = this.fileType.some((type) => {
          if (file.type.indexOf(type) > -1) return true
          if (fileExtension && fileExtension.indexOf(type) > -1) return true
          return false
        })

        if (!isTypeOk) {
          this.$message.error(`文件格式不正确, 请上传${this.fileType.join('/')}格式文件!`)
          return false
        }
      }

      // 校检文件大小
      if (this.fileSize) {
        const isLt = file.size / 1024 / 1024 < this.fileSize
        if (!isLt) {
          this.$message.error(`上传文件大小不能超过 ${this.fileSize} MB!`)
          return false
        }
      }

      // 校检相同文件
      if (!this.isCanUploadSame) {
        const isSame = this.list.some((item) => item.name + item.size === file.name + file.size)
        if (isSame) {
          this.$message.error(`此文件已上传!`)
          return false
        }
      }
      return true
    },
    // 文件个数超出
    handleExceed () {
      this.$message.error(`最多上传${this.limit}个文件`)
    },
    // 上传失败
    handleUploadError (err) {
      this.$message.error('上传失败, 请重试')
      this.$emit('error', err)
    },
    // 上传成功回调
    handleUploadSuccess (response, file) {
      this.$message.success('上传成功')
      if (this.responseFn) {
        response = this.responseFn(response, file, this.list)
      }
      if (this.multiple) {
        this.$emit('input', [...this.list, response])
      } else {
        this.$emit('input', response)
      }
    },
    // 删除
    handleRemove(index) {
      if (this.multiple) {
        const data = [...this.list]
        data.splice(index, 1)
        this.$emit('input', data || [])
      } else {
        this.$emit('input', {})
      }
    }
  }
}
</script>