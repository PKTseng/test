# user
```javascript=
<template>
  <div class="app-container">
    <div class="filter-container">
      <el-button
        type="primary"
        class="filter-item"
        icon="el-icon-edit"
        @click="handleCreate"
      >{{ $t('user.addUser') }}</el-button>
    </div>

    <!-- ---Create User Info--- -->
    <el-dialog
      :title="$t('user.addUser')"
      :visible.sync="dialogFormVisible"
      width="35%"
    >
      <el-form
        ref="createForm"
        :model="createForm"
        :rules="rules"
        @submit.prevent.native
      >
        <el-form-item
          :label="$t('userName')"
          prop="username"
        >
          <el-input v-model.trim="createForm.username" />
        </el-form-item>
        <el-form-item
          :label="$t('password')"
          prop="password"
        >
          <el-input
            v-model.trim="createForm.password"
            show-password
          />
        </el-form-item>
        <el-form-item
          :label="$t('role')"
          prop="role"
        >
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
        <el-button @click="closeForm()">{{ $t('close') }}</el-button>
        <el-button
          type="primary"
          @click="handleCreateUser()"
        >{{ $t('confirm') }}</el-button>
      </div>
    </el-dialog>
    <!-- ---Create User Info--- -->

    <!-- ---User Information Table--- -->
    <el-table
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%;"
      v-bind="$store.state.settings.table.attrs"
    >
      <el-table-column
        prop="username"
        :label="$t('userName')"
        align="center"
      />
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
          <el-button @click.native.prevent="handleDelete($index, row)">
            {{ $t('delete') }}
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
      list: null
    }
  },
  computed: {
    rules() {
      return {
        username: [
          {
            required: true,
            message: this.$t('user.formRules.userName_Empty'),
            trigger: 'blur'
          },
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
  created() {
    this.getAllUser()
  },
  methods: {
    validatePassword(rule, value, callback) {
      if (!this.checkPassword(value)) {
        callback(new Error(this.$t('user.formRules.password_CharType')))
      } else {
        return callback()
      }
    },
    validateSpace(rule, value, callback) {
      var reg = /^[^\s]*$/
      if (!reg.test(value)) {
        callback(new Error(this.$t('user.formRules.userName_Space')))
      } else {
        return callback()
      }
    },
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
    getAllUser() {
      this.listLoading = true
      getAllUser()
        .then((response) => {
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
      this.$refs['createForm'].validate((valid) => {
        if (valid) {
          this.$confirm(this.$t('createWarning'), this.$t('warning'), {
            confirmButtonText: this.$t('yes'),
            cancelButtonText: this.$t('cancel'),
            type: 'warning'
          })
            .then(() => {
              createUser(this.createForm)
                .then((response) => {
                  this.createForm.id = response.results.id
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
    handleDelete(index, row) {
      this.$confirm(this.$t('deleteWarning'), this.$t('warning'), {
        confirmButtonText: this.$t('yes'),
        cancelButtonText: this.$t('no'),
        type: 'warning'
      })
        .then(() => {
          deleteUser(row.id)
          this.list.splice(index, 1)
          this.tempIndex = undefined
          this.$notify({
            title: this.$t('success'),
            message: `${this.$t('deleteMsg')} ( ${row.username} )`,
            type: 'success',
            duration: 3000
          })
        })
        .catch(() => {})
    },
    cancelEdit(row) {
      row.edit = false
    },
    closeForm() {
      this.dialogFormVisible = false
    }
  }
}
</script>

```