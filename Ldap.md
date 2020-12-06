# Ldap
```javascript
<template>
  <div>
    <h1 class="page-title">{{ $t('settings.ldap.title') }}</h1>
    <div
      v-loading.fullscreen.lock="fullscreenLoading"
      v-loading="loading"
      class="app-container"
    >
      <el-card
        class="box-card"
        style="max-width: 80%"
      >
        <div
          slot="header"
          class="clearfix"
        >
          <span>{{ $t('settings.ldap.cardTitle') }}</span>
        </div>
        <el-form
          ref="dataForm"
          :rules="rules"
          :model="formData.config"
          label-position="left"
          label-width="20%"
          :disabled="formDisabled"
          @submit.prevent.native
        >
          <el-form-item>
            <el-checkbox v-model="formData.config.enable">
              {{ $t('settings.ldap.form.enable') }}
            </el-checkbox>
          </el-form-item>
          <div v-if="formData.config.enable">
            <div
              v-for="item in monitoringSettingsCols"
              :key="item.name"
            >
              <el-form-item
                :label="$t(`settings.ldap.form.${item.name}`)"
                :prop="item.name"
              >
                <el-input
                  v-model.trim="formData.config[item.name]"
                  :placeholder="item.placeholder"
                  clearable
                  :show-password="item.name === 'bind_password'"
                  @keyup.enter.native="handleUpdateLdapSettings"
                />
              </el-form-item>
            </div>
          </div>
        </el-form>
        <el-button
          class="button button-small button-border button-rounded"
          style="align: right"
          @click="handleUpdateLdapSettings"
        >
          {{ $t('save') }}
        </el-button>
      </el-card>
    </div>
  </div>
</template>

<script>
import {
  getLdapSettings,
  updateLdapSettings,
  checkReload,
  setupLdapSettings,
  reloadSettings,
  checkHealthStatus
} from '@/api/settings'

export default {
  name: 'LDAP',
  data() {
    return {
      formDisabled: false,
      loading: true,
      fullscreenLoading: false,
      healthStates: true,
      formData: {
        config: {},
        id: ''
      },
      monitoringSettingsCols: [
        {
          name: 'server_uri',
          placeholder: 'ldap://192.168.1.1'
        },
        {
          name: 'bind_dn',
          placeholder: 'CN=test,OU=employee,DC=example,DC=com'
        },
        {
          name: 'bind_password',
          placeholder: 'password'
        },
        {
          name: 'user_search',
          placeholder: 'OU=employee,DC=example,DC=com'
        },
        {
          name: 'group_search',
          placeholder: 'OU=org,DC=example,DC=com'
        },
        {
          name: 'group_type',
          placeholder: 'CN'
        }
      ]
    }
  },
  computed: {
    rules() {
      return {
        server_uri: [
          {
            required: true,
            message: this.$t('settings.ldap.formRules.server_uri'),
            trigger: 'blur'
          }
        ],
        bind_dn: [
          {
            required: true,
            message: this.$t('settings.ldap.formRules.bind_dn'),
            trigger: 'blur'
          }
        ],
        bind_password: [
          {
            required: true,
            message: this.$t('settings.ldap.formRules.bind_password'),
            trigger: 'blur'
          }
        ],
        user_search: [
          {
            required: true,
            message: this.$t('settings.ldap.formRules.user_search'),
            trigger: 'blur'
          }
        ],
        group_search: [
          {
            required: true,
            message: this.$t('settings.ldap.formRules.group_search'),
            trigger: 'blur'
          }
        ],
        group_type: [
          {
            required: true,
            message: this.$t('settings.ldap.formRules.group_type'),
            trigger: 'blur'
          }
        ]
      }
    }
  },
  created() {
    this.handleGetLdapSettings()
  },
  methods: {
    handleGetLdapSettings() {
      getLdapSettings()
        .then((response) => {
          this.formData = response[0]
        })
        .finally(() => {
          this.loading = false
        })
    },
    handleUpdateLdapSettings() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.formDisabled = true
          this.$confirm(this.$t('saveWarning'), this.$t('warning'), {
            confirmButtonText: this.$t('yes'),
            cancelButtonText: this.$t('no'),
            type: 'warning'
          })
            .then(() => {
              this.fullscreenLoading = true
              this.handelCheckReload()
            })
            .finally(() => {
              this.formDisabled = false
            })
        }
      })
    },
    handelCheckReload() {
      checkReload()
        .then((response) => {
          if (response.results.can_reload) {
            this.handelUpdateLdapSettings()
          } else {
            this.$notify({
              title: this.$t('warning'),
              message: response.log,
              type: 'warning',
              duration: 6000
            })
            this.fullscreenLoading = false
          }
        })
        .catch(() => {
          this.fullscreenLoading = false
        })
    },
    handelUpdateLdapSettings() {
      updateLdapSettings(this.formData, this.formData.id)
        .then((response) => {
          this.$notify({
            title: this.$t('success'),
            message: this.$t('updateMsg'),
            type: 'success',
            duration: 6000
          })
          this.handelSetupLdapSettings()
        })
        .catch(() => {
          this.fullscreenLoading = false
        })
    },
    handelSetupLdapSettings() {
      setupLdapSettings()
        .then((response) => {
          if (response.results.reload) {
            this.handelReloadSettings()
          } else {
            this.$notify({
              title: this.$t('success'),
              message: response.log,
              type: 'success',
              duration: 6000
            })
            this.fullscreenLoading = false
          }
        })
        .catch(() => {
          this.fullscreenLoading = false
        })
    },
    handelReloadSettings() {
      reloadSettings()
        // eslint-disable-next-line space-before-function-paren
        .then(async (response) => {
          let status = false
          for (let i = 0; i < 3; i++) {
            await this.handelCheckHealthStatus()
            if (this.healthStates) {
              status = true
              break
            }
          }
          if (!status) {
            this.$notify({
              title: this.$t('warning'),
              message: response.log,
              type: 'warning',
              duration: 6000
            })
          }
          this.fullscreenLoading = false
        })
        .catch(() => {
          this.fullscreenLoading = false
        })
    },
    handelCheckHealthStatus() {
      return new Promise((resolve) => {
        setTimeout(() => {
          checkHealthStatus()
            .then((response) => {
              this.healthStates = true
              for (const key in response.results) {
                if (!response.results[key]) {
                  this.healthStates = false
                  break
                }
              }
              resolve()
            })
            .catch(() => {
              this.fullscreenLoading = false
            })
        }, 3000)
      })
    }
  }
}
</script>

```