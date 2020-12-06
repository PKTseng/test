
# License
```javascript
<template>
  <div>
    <h1 class="page-title">{{ $t('route.license') }}</h1>
    <div
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
          <span>{{ $t('settings.license.title') }}</span>
        </div>
        <el-form
          v-if="!loading"
          ref="dataForm"
          :rules="rules"
          :model="formData"
          label-position="left"
          label-width="20%"
          :disabled="formDisabled"
          @submit.prevent.native
        >
          <el-form-item
            :label="$t(`settings.license.form.licenseKey`)"
            prop="key"
          >
            <el-input
              v-model.trim="formData.key"
              :placeholder="$t(`settings.license.formPlaceholder.licenseKey`)"
              maxlength="16"
              show-word-limit
              clearable
              @keyup.enter.native="handelUpdateFeaturesLicense"
            />
          </el-form-item>
        </el-form>
        <el-button
          class="button button-small button-border button-rounded"
          style="align: right"
          @click="handelUpdateFeaturesLicense"
        >
          {{ $t('save') }}
        </el-button>
      </el-card>
    </div>
  </div>
</template>

<script>
import { getFeaturesLicense, updateFeaturesLicense } from '@/api/featuresLicense'
export default {
  name: 'License',
  data() {
    return {
      formDisabled: false,
      loading: false,
      formData: {
        key: ''
      },
      licenseId: ''
    }
  },
  computed: {
    rules() {
      return {
        key: [
          {
            required: true,
            message: this.$t('settings.license.formRules.shouldNotBeBlack'),
            trigger: 'blur'
          },
          {
            min: 16,
            max: 16,
            message: this.$t('settings.license.formRules.stringLengthMustBe'),
            trigger: 'blur'
          }
        ]
      }
    }
  },
  created() {
    this.handelGetFeaturesLicense()
  },
  methods: {
    handelGetFeaturesLicense() {
      getFeaturesLicense().then((res) => {
        this.formData.key = res[0].key
        this.licenseId = res[0].id
      })
    },
    handelUpdateFeaturesLicense() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.formDisabled = true
          this.$confirm(this.$t('saveWarning'), this.$t('warning'), {
            confirmButtonText: this.$t('yes'),
            cancelButtonText: this.$t('no'),
            type: 'warning'
          })
            .then(() => {
              updateFeaturesLicense(this.licenseId, this.formData).then((response) => {
                this.$notify({
                  title: this.$t('success'),
                  message: this.$t('updateMsg'),
                  type: 'success',
                  duration: 6000
                })
              })
            })
            .finally(() => {
              this.formDisabled = false
            })
        }
      })
    }
  }
}
</script>
```