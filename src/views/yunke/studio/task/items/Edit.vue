<template>
<div>
    <el-dialog
    :title="title"
    :width="width"
    top="3vh"
    :close-on-click-modal="false" 
    :close-on-press-escape="false"
    :visible.sync="isVisible"
  >
    <el-form ref="form" :model="tasks" :rules="rules" label-position="right" label-width="100px">
      <el-form-item label="标题" prop="title">
        <el-input v-model="tasks.title"  />
      </el-form-item>
      <el-form-item label="摘要">
        <el-input v-model="tasks.introduction"  />
      </el-form-item>
       <el-form-item label="开始时间">
        <el-date-picker
          v-model="tasks.startTime"
          type="date"
          value-format="yyyy-MM-dd"
          placeholder="选择日期"
          style="width:100%"
          >
        </el-date-picker>
      </el-form-item>
      <el-form-item label="结束时间">
        <el-col>
          <el-date-picker
            v-model="tasks.endTime"
            type="date"
            value-format="yyyy-MM-dd"
            placeholder="选择日期"
            style="width:100%"
            >
          </el-date-picker>
        </el-col>
      </el-form-item>
      <el-form-item label="花费">
        <el-input v-model.number="tasks.cost" />
      </el-form-item>
      <el-form-item :label="$t('table.user.status')" prop="state">
        <el-radio-group v-model="tasks.state">
          <el-radio :label="1">进行中</el-radio>
          <el-radio :label="2">已完成</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="报销" >
        <el-radio-group v-model="tasks.reimbursement">
          <el-radio :label="0">未报销</el-radio>
          <el-radio :label="1">已报销</el-radio>
        </el-radio-group>
      </el-form-item>
    <el-form-item label="负责人" >
      <el-select  v-model="team.reliable"  value="" placeholder="负责人" style="width:100%">
          <el-option
            v-for="item in userRoles"
            :key="item.userId"
            :label="item.fullName"
            :value="String(item.userId)"
          />
        </el-select>
    </el-form-item>
    <el-form-item label="成员" >
      <el-select  v-model="team.member" multiple value="" placeholder="成员(不包含负责人)" style="width:100%">
        <el-option
          v-for="item in userRoles"
          :key="item.userId"
          :label="item.fullName"
          :value="String(item.userId)"
        />
      </el-select>
    </el-form-item>
    <el-form-item label="指导老师" >
      <el-select  v-model="team.teacher" multiple value="" placeholder="指导老师" style="width:100%">
        <el-option
          v-for="item in teacherRoles"
          :key="item.userId"
          :label="item.fullName"
          :value="String(item.userId)"
        />
      </el-select>
    </el-form-item>
      <el-form-item label="发票" prop="invoice">
        <!-- <el-button size="small" type="primary" @click="upload('发票')">点击上传</el-button> -->
        <el-upload
          :before-upload="handleBeforeUploadInvoice"
          :before-remove="addInvoiceBeforeRemove"
          :on-success="handleSuccessInvoice"
          :file-list="invoiceFileList"
          :action="uploadUrl"
          :class="{hideupload:editrepairinvoicehideupload, picturecard:true}"
          accept=".jpg,.jpeg,.png,.gif,.bmp"
          :headers="headers"
          multiple
          list-type="picture-card"
          :limit="uploadPicLimit"
          drag
          :on-preview="handlePreview"
        >
          <i  class="el-icon-upload" />
          <div slot="tip" style="display: block;" class="el-upload__tip">请勿上传违法文件，可同时上传3个附件，且文件不超过5M</div>
        </el-upload>
      </el-form-item>
      <el-form-item label="项目说明书" prop="specification">
       <!-- <el-input v-model="tasks.ce" /> -->
       <!-- <el-button size="small" type="primary" @click="upload('项目说明书')">点击上传</el-button> -->
         <el-upload
          :before-upload="handleBeforeUploadSpecif"
          :before-remove="handleBeforeRemove"
          :on-success="handleSuccessSpecification"
          :file-list="fileList"
          :action="uploadUrl"
          accept=".doc,.docx,.pdf,.ppt"
          :class="{hideupload:editrepairinvoicehideupload, picturecard:true}"
          :headers="headers"
          :limit="uploadDocTarLimit"
          drag
        >
          <i  class="el-icon-upload" />
          <div slot="tip" style="display: block;" class="el-upload__tip">请勿上传违法文件，可同时上传1个附件，且文件不超过5M</div>
        </el-upload>
       </el-form-item> 
       <el-form-item label="源文件" prop="url">
        <el-input v-model="tasks.url" />
        <!-- <el-upload
          :before-upload="handleBeforeUploadUrl"
          :before-remove="handleBeforeRemove"
          :on-success="handleSuccessUrl"
          :file-list="fileList"
          :action="uploadUrl"
          accept=".doc,.pdf,.docx,.ppt,.rar,.zip,.arj,.gz,.tar,.tar.gz"
          :class="{hideupload:editrepairinvoicehideupload, picturecard:true}"
          :headers="headers"
          :limit="uploadDocTarLimit"
          list-type="picture-card"
          drag
        >
          <i  class="el-icon-upload" />
          <div slot="tip" style="display: block;" class="el-upload__tip">请勿上传违法文件，可同时上传1个附件，且文件不超过10M</div>
        </el-upload> -->
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button type="primary" plain :loading="buttonLoading" @click="isVisible = false">
        {{ $t('common.cancel') }}
      </el-button>
      <el-button type="primary" plain :loading="buttonLoading" @click.prevent="submitForm">
        {{ $t('common.confirm') }}
      </el-button>
    </div>
  </el-dialog>
  <!-- 图片预览 -->
  <el-dialog title="图片预览" :visible.sync="previewVisible" width="40%" @close="previewDialogClose">
    <el-image :src="previewPath" alt class="previewImg" />
    </el-dialog>
</div>
</template>
<script>
import '@riophae/vue-treeselect/dist/vue-treeselect.css'
import { qiNiuUrl } from '@/settings'
import { getToken } from '@/utils/auth'
import {validateFileDocmentExt, validateFileExt, validatePicExt, validateFileTarExt } from '@/utils/my-validate'

export default {
  name: 'ItemsEdit',
  props: {
    dialogVisible: {
      type: Boolean,
      default: false
    },
    title: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      // 上传图片数量限制
      uploadPicLimit: 3,
      // 上传文档和压缩包数量限制
      uploadDocTarLimit: 1,
      // 超过图片数量限制时隐藏上传组件
      editrepairinvoicehideupload: false,
      headers: {
        Authorization: `bearer ${getToken()}`
      },
      uploadUrl: qiNiuUrl,
      initFlag: false,
      // 原始图片数据
      fileList: [],
      files: [],
      // 发票上传的数据
      invoiceFiles: [],
      invoiceFileList: [],
      addInvoiceListLength: 0,
      // 保存图片地址
      fileUrlList: [],
      invoiceFileUrlList: [],
      tasks: this.initTasks(),
      buttonLoading: false,
      screenWidth: 0,
      width: this.initWidth(),
      userRoles: [],
      teacherRoles: [],
      rules: {
        title: [
          { required: true, message: this.$t('rules.require'), trigger: 'blur' },
          { min: 2, max: 20, message: '长度为2-20', trigger: 'blur' }],
        state: { required: true, message: this.$t('rules.require'), trigger: 'blur' }
      },
      // 负责人，成员，指导老师
      team: {
        reliable: '',
        member: [],
        teacher: []
      },
      // 保存预览图片路径
      previewPath: '',
      //预览窗口显示与影藏
      previewVisible: false,
    }
  },
  computed: {
    isVisible: {
      get() {
        return this.dialogVisible
      },
      set() {
        this.close()
        this.reset()
      }
    }
  },
  mounted() {
    this.initUserRoles()
    window.onresize = () => {
      return (() => {
        this.width = this.initWidth()
      })()
    }
  },
  methods: {
    initTasks() {
      return {
        itemsId: '',
        title: '',
        introduction: '',
        startTime: '',
        endTime: '',
        cost: 0.0,
        invoice: '',
        specification: '',
        url: '',
        state: 1,
        reimbursement: 0,
        userId: '',
        m_state: []
      }
    },
    initWidth() {
      this.screenWidth = document.body.clientWidth
      if (this.screenWidth < 991) {
        return '90%'
      } else if (this.screenWidth < 1400) {
        return '45%'
      } else {
        return '800px'
      }
    },
    initUserRoles() {
      this.$get('system/user').then((r) => {
        console.log(11, r)
        const userRoles = []
        const teacherRoles = []
        const rows = r.data.data.rows
        rows.forEach((v, i) => {
          const obj = {
            userId: v.userId,
            fullName: v.fullName
          }
          if (v.noteId === '1') teacherRoles.push(obj)
          else if (v.noteId === '2' || v.noteId === '3') userRoles.push(obj)
        })
        this.userRoles = userRoles
        this.teacherRoles = teacherRoles
      }).catch((error) => {
        console.error(error)
        this.$message({
          message: this.$t('tips.getDataFail'),
          type: 'error'
        })
      })
    },
    setTasks(val) {
      // 获得数据
      // 浅克隆，同一源里的数值也会改变
      // this.tasks = row;
      this.tasks = Object.assign({}, val)
      this.team.reliable = val.reliable
      this.team.member = val.member
      this.team.teacher = val.teacher
      if (this.tasks.invoice.length > 1) {
        this.invoiceFileUrlList = this.tasks.invoice.split(',')
      } else {
        this.invoiceFileUrlList = this.tasks.invoice
      }
      for (var i = 0; i < this.invoiceFileUrlList.length; i++) {
        var fileurl = this.invoiceFileUrlList[i]
        if (fileurl !== null || fileurl !== '') {
          this.invoiceFileList.push({
            name: fileurl.substring(28),
            url: fileurl
          })
        }
      }
    },
    close() {
      this.team = {
        reliable: '',
        member: [],
        teacher: []
      }
      this.doSubmit()
      this.$emit('close')
    },
    submitForm() {
      this.$refs.form.validate((valid) => {
        if (valid) {
          this.buttonLoading = true
          if (!this.tasks.itemsId) {
            // create
            // 调用getDge
            const tasks = this.tasks
            const { a, b, flag } = this.getDge(this.team.reliable, this.team.member, this.team.teacher)
            if (flag) {
              this.buttonLoading = false
              return this.$message.info('不能多次选择同一个人，只能选择一次')
            }
            this.doSubmit()
            tasks.userId = a
            tasks.m_state = b
            this.$post('studio/items', { ...tasks }).then(() => {
              this.buttonLoading = false
              this.isVisible = false
              this.$message({
                message: this.$t('tips.createSuccess'),
                type: 'success'
              })
              this.$emit('success')
              this.team = {
                reliable: '',
                member: [],
                teacher: []
              }
            })
          } else {
            // update
            // 调用getDge
            const tasks = this.tasks
            const { a, b, flag } = this.getDge(this.team.reliable, this.team.member, this.team.teacher)
            if (flag) {
              this.buttonLoading = false
              return this.$message.info('不能多次选择同一个人，只能选择一次')
            }
            tasks.userId = a
            tasks.m_state = b
            delete tasks.members
            console.log(22, tasks)
            const updateState = {
              state: tasks.state,
              itemsId: tasks.itemsId
            }
            this.$put('studio/items/state', { ...updateState })
            this.$put('studio/items', { ...tasks }).then((r) => {
              this.buttonLoading = false
              this.isVisible = false
              this.$message({
                message: this.$t('tips.updateSuccess'),
                type: 'success'
              })
              this.$emit('success')
            })
          }
        } else {
          return false
        }
      })
    },
    getDge(reliable, member, teacher) {
      const reliableArr = []
      const memberArr = []
      const teacherArr = []
      // 负责人
      if (reliable.length > 0) {
        reliableArr.push('1')
      }
      // 成员
      member.forEach((v, i) => {
        memberArr.push('2')
      })
      // 指导老师
      teacher.forEach((v, i) => {
        teacherArr.push('3')
      })
      let a = [...reliable, ...member, ...teacher]
      const b = [...reliableArr, ...memberArr, ...teacherArr].join(',')
      // 进行判断是否多次选择同一个人
      let flag = false
      var obj = {}
      for (var i = 0; i < a.length; i++) {
        if (obj[a[i]]) flag = true
        obj[a[i]] = true
      }
      a = a.join(',')
      return {
        a,
        b,
        flag
      }
    },
    reset() {
      // 先清除校验，再清除表单，不然有奇怪的bug
      this.$refs.form.clearValidate()
      this.$refs.form.resetFields()
      this.tasks = this.initTasks()
    },
    handleSuccessInvoice(response, file, fileList) {
      const uid = file.uid
      const id = response.data.contentId
      this.invoiceFiles.push({ uid, id })
      this.addInvoiceListLength++;
      if (this.tasks.invoice === '' || this.tasks.invoice === null) {
        this.tasks.invoice = response.data.url
      } else {
        this.tasks.invoice = this.tasks.invoice + ',' + response.data.url
      }
      // this.editInvoiceFileList.push({
      //   name: response.data.url.substring(28),
      //   url: response.data.url,
      // });
    },
    handleSuccessSpecification(response, file, fileList) {
      const uid = file.uid
      const id = response.data.contentId
      this.files.push({ uid, id })
      if (this.tasks.specification === '' || this.tasks.specification === null) {
        this.tasks.specification = response.data.url
      } else {
        this.tasks.specification = this.tasks.specification + ',' + response.data.url
      }
    },
    // handleSuccessUrl(response, file, fileList) {
    //   const uid = file.uid
    //   const id = response.data.contentId
    //   this.files.push({ uid, id })
    //   if (this.tasks.url === '' || this.tasks.url === null) {
    //     this.tasks.url = response.data.url
    //   } else {
    //     this.tasks.url = this.tasks.url + ',' + response.data.url
    //   }
    // },
    // 删除添加表单上传的发票图片，写死的（图片长度限定为3）
    addInvoiceBeforeRemove (file, fileList) {
      let fileUrl = ''
      let flag = false
      // 添加
      if (this.title === '新增') {
         fileUrl = file.response.data.url
         flag = true
       } else {
         // 修改
         fileUrl = file.url
       }
      console.log(file,this.invoiceFileList)
      if(flag) this.addInvoice(fileUrl, file)
      else this.updateInvoice(file)
       
    },
    addInvoice(fileUrl, file) {
      for (let j = 0; j < this.invoiceFiles.length; j++) {
        if (this.invoiceFiles[j].uid === file.uid) {
          this.$delete(`oss/content/${this.invoiceFiles[j].id}`)
          // 根据图片数量分别执行删除的功能
          if (this.invoiceFiles.length === 1) {
            console.log('111')
            this.tasks.invoice = ''
          } else if (this.invoiceFiles.length === 2) {
            console.log('222');
            if (j === 0) {
              this.tasks.invoice = this.tasks.invoice.split(
                fileUrl + ','
              )[1]
            } else {
              this.tasks.invoice = this.tasks.invoice.split(
                ',' + fileUrl
              )[0]
            }
          } else {
            if (j === 0) {
              this.tasks.invoice = this.tasks.invoice.split(
                fileUrl + ','
              )[1];
            } else if (j === 1) {
              const firsturl = this.tasks.invoice.split(',' + fileUrl)[0]
              const lasturl = this.tasks.invoice.split(',' + fileUrl)[1]
              this.tasks.invoice = firsturl + lasturl
            } else {
              this.tasks.invoice = this.tasks.invoice.split(
                ',' + fileUrl
              )[0]
            }
          }
          this.addInvoiceListLength--;
          console.log(this.addInvoiceListLength)
          // this.addInvoiceChange();
          return true
        }
      }
    },
    updateInvoice(file) {
      for (let n = 0; n < this.invoiceFileList.length; n++) {
        if (this.invoiceFileList[n].uid === file.uid) {
          const fileUrl = this.invoiceFileList[n].url;
          const fileName = this.invoiceFileList[n].url
            .substring(28)
            .split('.')[0];
          this.$delete(`oss/content`, { fileName: fileName });
          // 根据图片数量分别执行删除的功能
          if (this.invoiceFileList.length === 1) {
            console.log('111');
            this.tasks.invoice = '';
            this.invoiceFileList = [];
          } else if (this.invoiceFileList.length === 2) {
            console.log('222');
            if (n === 0) {
              this.tasks.invoice = this.tasks.invoice.split(
                fileUrl + ','
              )[1];
              this.invoiceFileList.shift();
            } else {
              this.tasks.invoice = this.tasks.invoice.split(
                ',' + fileUrl
              )[0];
              this.invoiceFileList.pop();
            }
          } else {
            if (n === 0) {
              this.tasks.invoice = this.tasks.invoice.split(
                fileUrl + ','
              )[1];
              this.invoiceFileList.shift();
            } else if (n === 1) {
              const firsturl = this.tasks.invoice.split(',' + fileUrl)[0];
              const lasturl = this.tasks.invoice.split(',' + fileUrl)[1];
              this.tasks.invoice = firsturl + lasturl;
              this.invoiceFileList.splice(n, 1);
            } else {
              this.tasks.invoice = this.tasks.invoice.split(
                ',' + fileUrl
              )[0];
              this.invoiceFileList.pop();
            }
          }
          this.addInvoiceListLength--;
          // this.editInvoiceChange();
          return true;
        }
      }
    },
    handleBeforeRemove(file, fileList) {
      for (let i = 0; i < this.files.length; i++) {
        if (this.files[i].uid === file.uid) {
          this.$delete(`oss/content/${this.files[i].id}`).then(() => {
            this.$message({
              message: '删除成功',
              type: 'success'
            })
          })
          return true
        }
      }
    },
    handleBeforeUploadInvoice(file) {
      if (file.size / 1024 > 5000) {
        this.$message({
          message: '上传文件大小不能超过5MB!',
          type: 'error'
        })
        return false
      } else {
        const ext = file.name.replace(/.+\./, '')
        if (!validatePicExt(ext)) {
          this.$message({
            type: 'error',
            message: '禁止上传' + ext + '类型的附件'
          })
          return false
        }
      }
    },
    handleBeforeUploadSpecif(file) {
      if (file.size / 1024 > 5000) {
        this.$message({
          message: '上传文件大小不能超过5MB!',
          type: 'error'
        })
        return false
      } else {
        const ext = file.name.replace(/.+\./, '')
        if (!validateFileExt(ext)) {
          this.$message({
            type: 'error',
            message: '禁止上传' + ext + '类型的附件'
          })
          return false
        }
      }
    },
    // handleBeforeUploadUrl(file) {
    //   if (file.size / 1024 > 10000) {
    //     this.$message({
    //       message: '上传文件大小不能超过10MB!',
    //       type: 'error'
    //     })
    //     return false
    //   } else {
    //     const ext = file.name.replace(/.+\./, '')
    //     if (!validateFileExt(ext)) {
    //       this.$message({
    //         type: 'error',
    //         message: '禁止上传' + ext + '类型的附件'
    //       })
    //       return false
    //     }
    //   }
    // },
    // 刷新上传列表
    doSubmit() {
      this.fileList = []
      this.fileUrlList = []
      this.invoiceFileList = [];
      this.invoiceFiles = [];
      this.dialogImageUrl = ''
      this.dialog = false
    },
    // 修改时的图片预览效果
    handlePreview(file) {
      console.log(file)
      if ('url' in file) {
        this.previewPath = file.url
      } else {
        this.previewPath = file.response.data.url
      }
      this.previewVisible = true
    },
    // 图片预览关闭
    previewDialogClose() {
      this.previewPath = ''
      this.previewVisible = false
    }
  }
}
</script>
<style lang="scss" scoped>
</style>
