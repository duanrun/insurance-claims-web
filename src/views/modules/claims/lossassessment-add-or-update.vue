<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="交警定责（这里就是指判断双方责任比如46、37等）" prop="lossAssessmentFix duty">
      <el-input v-model="dataForm.lossAssessmentFixDuty" placeholder="交警定责（这里就是指判断双方责任比如46、37等）"></el-input>
    </el-form-item>
    <el-form-item label="保险人员（指的是负责定损的人员）" prop="lossAssessmentFix dutyUser">
      <el-input v-model="dataForm.lossAssessmentFixDutyUser" placeholder="保险人员（指的是负责定损的人员）"></el-input>
    </el-form-item>
    <el-form-item label="赔款id" prop="indemnityId">
      <el-input v-model="dataForm.indemnityId" placeholder="赔款id"></el-input>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          clossAssessmentId: 0,
          lossAssessmentFixDuty: '',
          lossAssessmentFixDutyUser: '',
          indemnityId: ''
        },
        dataRule: {
          lossAssessmentFixDuty: [
            { required: true, message: '交警定责（这里就是指判断双方责任比如46、37等）不能为空', trigger: 'blur' }
          ],
          lossAssessmentFixDutyUser: [
            { required: true, message: '保险人员（指的是负责定损的人员）不能为空', trigger: 'blur' }
          ],
          indemnityId: [
            { required: true, message: '赔款id不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.clossAssessmentId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.clossAssessmentId) {
            this.$http({
              url: this.$http.adornUrl(`/claim/lossassessment/info/${this.dataForm.clossAssessmentId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.lossAssessmentFixDuty = data.lossAssessment.lossAssessmentFixDuty
                this.dataForm.lossAssessmentFixDutyUser = data.lossAssessment.lossAssessmentFixDutyUser
                this.dataForm.indemnityId = data.lossAssessment.indemnityId
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/claim/lossassessment/${!this.dataForm.clossAssessmentId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'clossAssessmentId': this.dataForm.clossAssessmentId || undefined,
                'lossAssessmentFixDuty': this.dataForm.lossAssessmentFixDuty,
                'lossAssessmentFixDutyUser': this.dataForm.lossAssessmentFixDutyUser,
                'indemnityId': this.dataForm.indemnityId
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
