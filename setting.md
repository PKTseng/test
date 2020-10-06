# setting
```javascript
<template>
  <div v-loading="loading">
    <el-card
      class="box-card"
      style="max-width: 60%"
    >
      <div
        slot="header"
        class="clearfix"
      />
      <el-form
        v-if="!loading"
        ref="dataForm"
        :rules="rules"
        :model="formData"
        label-position="left"
        label-width="40%"
        @submit.prevent.native
      >
        <el-form-item
          :label="$t(`settings.monitoringSettings.form.key`)"
          prop="licenseKey"
        >
          <el-input
            v-model.trim="formData.licenseKey"
            placeholder="Insert Key"
            maxlength="16"
            show-word-limit
            clearable
          />
        </el-form-item>
      </el-form>
      <el-button
        class="button button-small button-border button-rounded button-blue"
        style="align: right"
        @click="updateFeaturesLicenseFunc"
      >
        {{ $t('save') }}
      </el-button>
    </el-card>
  </div>
</template>

<script>
import { getFeaturesLicense, updateFeaturesLicense } from '@/api/settings'
export default {
  name: 'Settings',
  data() {
    return {
      loading: false,
      formData: {
        licenseKey: ''
      },
      licenseId: ''
    }
  },
  computed: {
    rules() {
      return {
        licenseKey: [
          {
            required: true,
            message: 'Should not be blank',
            trigger: 'blur'
          },
          {
            min: 16,
            max: 16,
            message: 'String length must be 16',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  created() {
    this.getFeaturesLicenseFunc()
  },
  methods: {
    getFeaturesLicenseFunc() {
      getFeaturesLicense()
        .then((res) => {
          console.log(res)
          this.formData.licenseKey = res[0].key
          this.licenseId = res[0].id
        })
    },
    updateFeaturesLicenseFunc() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.$confirm(this.$t('saveWarning'), this.$t('warning'), {
            confirmButtonText: this.$t('yes'),
            cancelButtonText: this.$t('cancel'),
            type: 'warning'
          }).then(() => {
            const data = { key: this.formData.licenseKey }
            updateFeaturesLicense(this.licenseId, data)
              .then((response) => {
                console.log('update')
                console.log(response)
                this.$notify({
                  title: this.$t('success'),
                  message: this.$t('updateMsg'),
                  type: 'success'
                })
              })
              // .catch(() => {})
          })
        }
      })
    }
  }
}
</script>

```