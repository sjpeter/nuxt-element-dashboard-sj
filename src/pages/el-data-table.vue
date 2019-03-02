<template>
  <div class="el-data-table">
    <!-- <img class="home-img" :src="$store.state.meta.homePageImg" alt="homeImg" /> -->
    <el-data-table v-bind="tableConfig"></el-data-table>

    <el-dialog :title="dialogTitle" :visible.sync="dialogVisible">
        <el-form-renderer :content="tableConfig.form" ref="dialogForms" :disabled="isView">
            <!-- @slot 额外的弹窗表单内容, 当form不满足需求时可以使用  -->
            <slot name="form"></slot>
        </el-form-renderer>

        <div slot="footer" v-show="!isView">
            <el-button @click="cancel" size="small">取 消</el-button>
            <el-button type="primary" @click="confirm" v-loading="confirmLoading" size="small">确 定</el-button>
        </div>
    </el-dialog>
  </div>
</template>

<script>
//全局参数
const dialogForm = 'dialogForms'

export default {
  props: {
    form: {
      type: Array,
      default() {
        return []
      }
    },
    /**
     * 修改提交时注入额外的参数
     */
    extraParams: {
      type: Object
    },
    /**
     * 在修改弹窗 点击确认时调用，返回Promise, 如果reject, 则不会发送新增/修改请求
     * 参数: (data, isNew) data为表单数据, isNew true 表示是新增弹窗, false 为 编辑弹窗
     */
    beforeConfirm: {
      type: Function,
      default() {
        return Promise.resolve()
      }
    }
  },
  data() {
    return {
      classify: [
        {
          value: '1',
          label: '前端组件'
        },
        {
          value: '2',
          label: '测试组件'
        },
        {
          value: '3',
          label: '分布式组件'
        },
        {
          value: '4',
          label: '数据组件'
        },
        {
          value: '5',
          label: '存储组件'
        }
      ],
      state: [
        {
          value: '0',
          label: '下架'
        },
        {
          value: '1',
          label: '上架'
        }
      ],
      tableConfig: {
        url: '/security/api/v1/list',
        hasEdit: false,
        extraButtons: [
          {
            text: '查看',
            atClick: this['getDetails'],
            type: "primary"
          },
          {
            text: '编辑',
            atClick: this['edit']
          },
          {
            text: '下架',
            atClick: this['changeState'],
            "show": (row) => { if (row.state == '1') { return true } else { return false } }
          },
          {
            text: '上架',
            atClick: this['changeState'],
            "show": (row) => { if (row.state == '0') { return true } else { return false } }
          }
        ],
        tableAttrs: {
          "row-class-name": ({row}) => { if (row.state == '0') { return '' } else { return 'green' } }
        },
        columns: [
          {
            type: 'selection'
          },
          {
            prop: 'name',
            label: '组件名称',
            "class-name": "default"
          },
          {
            prop: 'classify',
            "class-name": "default",
            label: '分类'
          },
          {
            prop: 'version',
            "class-name": "default",
            label: '版本'
          },
          {
            prop: 'language',
            "class-name": "default",
            label: '开发语言'
          },
          {
            prop: 'time',
            "class-name": "default",
            label: '最后更新时间'
          },
          {
            prop: 'state',
            label: '状态',
            "formatter": (row, column) => { if (row.state == '0') { return '下架' } else { return '上架' } }
          }
        ],
        searchForm: [
          {
            $type: 'input',
            $id: 'name',
            label: '组件名称',
            $el: {
              placeholder: '请输入',
            }
          },
          {
            $type: 'select',
            $id: 'classify',
            label: '分类',
            $options: [],
            $el: {
              placeholder: '请选择'
            }
          },
          {
            $type: 'select',
            $id: 'state',
            label: '状态',
            $options: [],
            $el: {
              placeholder: '请选择'
            }
          }
        ],
        form: [
          {
            $type: 'input',
            $id: 'name',
            label: '用户名',
            $el: {
              placeholder: '请输入',
              size: 'small'
            },
            rules: [
              {
                required: true,
                message: '请输入用户名',
                trigger: 'blur'
              }
            ]
          },
          {
            $type: 'select',
            $id: 'classify',
            label: '分类',
            $options: [],
            $el: {
              placeholder: '请选择'
            },
            rules: [
              {
                required: true,
                message: '请选择分类',
                trigger: 'blur'
              }
            ]
          },
          {
            $type: 'input',
            $id: 'version',
            label: '版本',
            $options: [],
            $el: {
              placeholder: '请输入'
            },
            rules: [
              {
                required: true,
                message: '请输入版本',
                trigger: 'blur'
              }
            ]
          },
          {
            $type: 'input',
            $id: 'language',
            label: '语言',
            $options: [],
            $el: {
              placeholder: '请输入'
            },
            rules: [
              {
                required: true,
                message: '请输入语言',
                trigger: 'blur'
              }
            ]
          }
        ]
      },
      //弹窗
      dialogTitle: '',
      dialogVisible: false,
      isNew: true,
      isEdit: false,
      isView: false,
      confirmLoading: false,
      // 要修改的那一行
      row: {}
    }
  },
  created() {
    //初始
    this.tableConfig.form[1]['$options'] = this.tableConfig.searchForm[1]['$options'] = this.classify
    this.tableConfig.form[2]['$options'] = this.tableConfig.searchForm[2]['$options'] = this.state
  },
  methods: {
    /**
    * 查看
    * row object
    */
    getDetails(row) {
      let that = this
      return new Promise((resolve, reject) => {
        that.row = row
        that.isView = true
        that.dialogTitle = '查看'
        that.dialogVisible = true

        // 给表单填充值
        that.$nextTick(() => {
          that.$refs[dialogForm].updateForm(row)
          resolve(true)
        })
      });
    },
    /**
    * 编辑
    * row object
    */
    edit(row) {
      let that = this
      return new Promise((resolve, reject) => {
        that.row = row
        that.isEdit = true
        that.isView = false
        that.dialogTitle = '查看'
        that.dialogVisible = true

        // 给表单填充值
        that.$nextTick(() => {
          that.$refs[dialogForm].updateForm(row)
          resolve(true)
        })
      });
    },
    /**
    * 改变状态
    * row object
    */
    changeState (row) {
      let that = this
      return new Promise((resolve, reject) => {
        this.$confirm('确认' + this.state[row.state ? '0' : '1']['label'] + '吗', '提示', {
          type: 'warning',
          beforeClose: (action, instance, done) => {
            if (action == 'confirm') {
              instance.confirmButtonLoading = true

              this.$axios
                .patch(this.tableConfig.url + '/' + row.id + '?state=' + row.state)
                .then(resp => {
                  instance.confirmButtonLoading = false
                  done()
                  resolve(true)
                  this.showMessage(resp.data.state)
                })
                .catch(er => {
                  instance.confirmButtonLoading = false
                })
            } else {
              done()
              resolve(true)
            }
          }
        }).catch(er => {
          /*取消*/
        })
      });
    },
    /**
    * 取消弹框
    */
    cancel() {
      this.dialogVisible = false
    },
    /**
    * 确认弹框
    */
    confirm() {
      if (this.isView) {
        this.cancel()
        return
      }

      this.$refs[dialogForm].validate(valid => {
        if (!valid) return false

        let data = Object.assign(
          {},
          this.$refs[dialogForm].getFormValue(),
          this.extraParams
        )

        this.beforeConfirm(data, this.isNew)
          .then(resp => {
            // 修改逻辑
            let method = 'post'
            let url = this.tableConfig.url + ''

            if (this.isEdit) {
              method = 'put'
              url += `/${this.row['id']}`
            }

            this.confirmLoading = true

            this.$axios[method](url, data)
              .then(resp => {
                this.showMessage(resp.data.state)
                this.cancel()
                this.confirmLoading = false
              })
              .catch(err => {
                this.confirmLoading = false
              })
          })
          .catch(e => {})
      })
    },
    /**
    * 消息
    */
    showMessage(isSuccess = true) {
      if (isSuccess) {
        this.$message({
          type: 'success',
          message: '操作成功'
        })
      } else {
        this.$message({
          type: 'error',
          message: '操作失败'
        })
      }
    }
  }
}
</script>
<style lang="stylus">
  .green{
    color: green;
  }
  .default{
    color: #606266;
  }
</style>
