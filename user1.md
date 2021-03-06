# user.vue
```javascript
<template>
  <div class="app-container">
    <div ref="filter-container" class="filter-container">
      <el-button
        v-permission="['Admin', 'Editor']"
        type="primary"
        icon="el-icon-edit"
        @click="handleCreate"
      >
        {{ $t('user.addUser') }}
      </el-button>
      <el-button
        :class="['control-delete button button-small button-border button-rounded', multipleSelection.length ? 'button-blue' : 'disabled']"
        type="primary"
        icon="el-icon-delete-solid"
        :disabled="multipleSelection.length === 0"
        @click="handleDelete"
      >
        {{ $t('delete') }}
      </el-button>
    </div>
    <!-- ---Create User Info--- -->
    <!-- 彈跳視窗 -->
    <!-- visible.sync 判定顯示彈跳式窗 -->
    <el-dialog
      :title="$t('user.addUser')"
      :visible.sync="dialogFormVisible"
      width="35%"
    >
      <!-- 套用規則 -->
      <!-- ref 是類似抓取某 DOM 的概念，只要在模板使用 ref
      ，同時在 srcipt 就會連帶使用 $refs[''] 來抓取，這是固定的! -->
      <el-form
        ref="createForm"
        :model="createForm"
        :rules="rules"
        @submit.prevent.native
      >
        <!-- 輸入欄位 -->
        <el-form-item
          :label="$t('userName')"
          prop="username"
        >
          <!-- 欄位輸入文字 -->
          <el-input v-model.trim="createForm.username" />
        </el-form-item>
        <!-- 密碼欄位 -->
        <el-form-item
          :label="$t('password')"
          prop="password"
        >
          <!-- 密碼欄位輸入文字 -->
          <el-input
            v-model.trim="createForm.password"
            show-password
          />
        </el-form-item>
        <el-form-item
          :label="$t('role')"
          prop="role"
        >
          <!-- 用 prop 綁的 role 跟 v-model 綁的 role 不一樣，一個是綁規矩一個是綁資料 -->
          <!-- 下拉選單 -->
          <el-select
            v-model="createForm.role"
            :placeholder="$t('user.selectRole')"
          >
            <el-option
              label="Admin"
              value="Admin"
            />
            <el-option
              label="Editor"
              value="Editor"
            />
            <el-option
              label="Viewer"
              value="Viewer"
            />
          </el-select>
        </el-form-item>
      </el-form>
      <div
        slot="footer"
        class="dialog-footer"
      >
        <!--()裡帶參數-->
        <el-button @click="closeForm()">{{ $t('close') }}</el-button>
        <el-button
          type="primary"
          @click="handleCreateUser()"
        >{{ $t('confirm') }}</el-button>
      </div>
    </el-dialog>
    <!-- ---Create User Info--- -->

    <!-- ---User Information Table--- -->
    <!-- listLoading 網頁讀取狀態 -->
    <!-- :data="list" ， API 打進來後會動態把資料綁到 list 上-->
    <!-- v-bind 動態綁定 $store.state.setting 指到$store.state src/store/midules?setting 資料夾下的setting -->
    <el-table
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%;"
      v-bind="$store.state.settings.table.attrs"
      @selection-change="handleSelectionChange"
    >
      <el-table-column
        type="selection"
        width="40"
        :selectable="canSelectRow"
      />
      <!-- userName 欄位第一欄 -->
      <el-table-column
        prop="username"
        :label="$t('userName')"
        align="center"
      />
      <!-- Role 欄位第二欄 -->
      <el-table-column
        prop="role"
        :label="$t('role')"
        align="center"
      >
        <template slot-scope="{row}">
          <template v-if="row.edit">
            <el-select
              v-model="updateForm.role"
              :placeholder="$t('user.selectRole')"
            >
              <el-option
                label="Admin"
                value="Admin"
              />
              <el-option
                label="Editor"
                value="Editor"
              />
              <el-option
                label="Viewer"
                value="Viewer"
              />
            </el-select>
            <el-button @click="cancelEdit(row)">{{ $t('cancel') }}</el-button>
          </template>
          <span v-else>{{ row.role }}</span>
        </template>
      </el-table-column>
      <!-- action 欄位第三欄 -->
      <el-table-column
        fixed="right"
        :label="$t('action')"
        align="center"
      >
        <template slot-scope="{row,$index}">
          <el-button
            v-if="row.edit"
            @click.native.prevent="updateData(row)"
          >
            {{ $t('ok') }}
          </el-button>
          <el-button
            v-else
            @click.native.prevent="handleUpdate($index, row)"
          >
            {{ $t('edit') }}
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- ---User Information Table--- -->
  </div>
</template>

<script>
import { getAllUser, createUser, deleteUser, editUser } from '@/api/user'

export default {
  name: 'User',
  data() {
    return {
      listLoading: true,
      dialogFormVisible: false,
      tempIndex: undefined,
      createForm: {
        // user: {
        //   email: ''
        // },
        username: '',
        password: '',
        role: '',
        id: undefined
      },
      updateForm: {
        id: undefined,
        username: '',
        role: ''
      },
      list: null,
      multipleSelection: []
    }
  },
  computed: {
    rules() {
      return {
        // username: {
        //   mail: [
        //     {
        //       required: true,
        //       message: this.$t('user.formRules.userName_Empty'),
        //       trigger: 'blur'
        //     }
        //   ]
        // },
        username: [
          {
            required: true,
            message: this.$t('user.formRules.userName_Empty'),
            trigger: 'blur'
          },
          // 下行為正規表達式驗證
          { validator: this.validateSpace, trigger: 'blur' }
        ],
        password: [
          {
            min: 8,
            message: this.$t('user.formRules.password_LessChar'),
            trigger: 'blur'
          },
          {
            required: true,
            message: this.$t('user.formRules.password_Empty'),
            trigger: 'blur'
          },
          { validator: this.validatePassword, trigger: 'blur' },
          { validator: this.validateSpace, trigger: 'blur' }
        ],
        role: [
          {
            required: true,
            message: this.$t('user.formRules.role_Empty'),
            trigger: ['blur', 'change']
          }
        ]
      }
    }
  },

  // 每個 component都有生命週期，但有些需要特別指定動作，例如以下是先打API
  // 先抓 getAllUser API
  created() {
    this.getAllUser()
  },

  // owen 解釋 refs 寫法
  // mounted() {
  // this.$refs['filter-container'].style.backgroundColor = 'red'
  // document.querySelector('.filter-container').style.backgroundColor = 'yellow'
  // },

  // validatePassword、validateSpace 輸入密碼跟正規表達式驗證
  methods: {
    validatePassword(rule, value, callback) {
      if (!this.checkPassword(value)) {
        callback(new Error(this.$t('user.formRules.password_CharType')))
      } else {
        return callback()
      }
    },
    validateSpace(rule, value, callback) {
      // /^[^\s]*$/ 正規表達式，限制使用者輸入帳密時比需依照規定填寫
      var reg = /^[^\s]*$/
      if (!reg.test(value)) {
        callback(new Error(this.$t('user.formRules.userName_Space')))
      } else {
        return callback()
      }
    },

    // checkPassword 正規表達式，限制寫法
    checkPassword(password) {
      var checkUpperCharacter = 0
      var checkLowerCharacter = 0
      var checkNoneCharacter = 0
      if (password.match(/([A-Z])+/)) {
        checkUpperCharacter++
      }
      if (password.match(/([a-z])+/)) {
        checkLowerCharacter++
      }
      if (password.match(/([0-9])+/)) {
        checkNoneCharacter++
      }
      if (password.match(/[^a-zA-Z0-9]+/)) {
        checkNoneCharacter++
      }
      if (checkUpperCharacter > 0 && checkLowerCharacter > 0 && checkNoneCharacter > 0) {
        return true
      }
    },
    // 兩個 getAllUser 是不一樣的
    // 外面的 getAllUser 是指這個 component 的 vue 實例裡面的 this.getAllUser
    getAllUser() {
      this.listLoading = true
      // 裡面的 getAllUser 是指外面 import 進來的 API
      getAllUser()
        .then((response) => {
          console.log(response.results)
          // map()會處理陣列中每個元素，最後回傳出一個新的陣列，裡頭有一個函式 ( 必填 )，v 是變數
          this.list = response.results.map((v) => {
            this.$set(v, 'edit', false)
            return v
          })
        })
        .finally((response) => {
          this.listLoading = false
        })
    },
    handleCreate() {
      this.dialogFormVisible = true
    },
    handleCreateUser() {
      // validate:表單驗證
      this.$refs['createForm'].validate((valid) => {
        // if true or false
        // alert(valid)
        if (valid) {
          this.$confirm(this.$t('createWarning'), this.$t('warning'), {
            confirmButtonText: this.$t('yes'),
            cancelButtonText: this.$t('cancel'),
            type: 'warning'
          })
          // 這邊用到 promise 概念
            .then(() => {
              // 引入API
              createUser(this.createForm)
                .then((response) => {
                  this.createForm.id = response.results.id
                  // 可用 ES6 寫法
                  // const tempData = (... this.createForm)
                  const tempData = Object.assign({}, this.createForm)
                  Object.assign(tempData, { edit: 'false' })
                  this.list.splice(this.list.length, 0, tempData)
                  this.list[this.list.length - 1].edit = false
                  this.dialogFormVisible = false
                  this.$notify({
                    title: this.$t('success'),
                    message: `${this.$t('createMsg')} ( ${this.createForm.username} )`,
                    type: 'success',
                    duration: 3000
                  })
                  this.$nextTick(() => {
                    this.$refs.createForm.resetFields()
                  })
                  this.$refs.createForm.resetFields()
                })
                .catch((response) => {
                  console.log(response)
                })
            })
            .catch(() => {})
        }
      })
    },
    handleUpdate(index, row) {
      this.updateForm = Object.assign({}, row)
      if (this.tempIndex !== undefined) {
        this.list[this.tempIndex].edit = false
      }
      this.tempIndex = index
      row.edit = true
    },
    updateData(row) {
      // const tempData = Object.assign({}, this.updateForm)
      const tempData = {
        id: this.updateForm.id,
        username: this.updateForm.username,
        role: this.updateForm.role
      }
      const index = this.list.findIndex((v) => v.id === this.updateForm.id)
      editUser(tempData).then((response) => {
        this.list.splice(index, 1, this.updateForm)
        row.edit = false
      })
      this.$notify({
        title: this.$t('success'),
        message: `${this.$t('updateMsg')} ( ${row.username} )`,
        type: 'success',
        duration: 3000
      })
    },
    handleDelete() {
      this.$confirm(this.$t('deleteWarning'), 'Notification', {
        confirmButtonText: this.$t('yes'),
        cancelButtonText: this.$t('cancel'),
        type: 'warning'
      })
        .then(() => {
          for (const ele of this.multipleSelection) {
            deleteUser(ele.id).then((response) => {
              // 判斷 ele.id 在list中的index位置
              const listIndex = this.list.findIndex(
                (item) => item.id === ele.id
              )
              this.list.splice(listIndex, 1)
              this.total--
            })
          }
          this.$notify({
            title: this.$t('succes'),
            message: this.$t('deleteMsg'),
            type: 'success',
            duration: 2000
          })
        })
        .catch(() => {})
    },
    handleSelectionChange(val) {
      console.log(val)
      this.multipleSelection = val
    },
    cancelEdit(row) {
      row.edit = false
    },
    closeForm() {
      this.dialogFormVisible = false
    },
    canSelectRow(row, index) {
      return row.username !== 'admin'
    }
  }
}
</script>
```