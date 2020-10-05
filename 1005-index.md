# 1005 index.vue
```html
<template>
  <div class="app-container">
    <el-tabs v-model="tabName">
      <el-tab-pane
        :label="$t('settings.globalAccount.title')"
        name="globalAccount"
      >
        <global-account-info-table
          :dialog="dialog"
          :list="list"
        />
        <global-account-dialog-form
          :dialog="dialog"
          :list="list"
        />
      </el-tab-pane>

      <el-tab-pane
        :label="$t('settings.monitoringSettings.title')"
        name="monitoringSetting"
      >
        <monitoring-settings-info-form :tab-name="tabName" />
      </el-tab-pane>

      <!-- 新增的License -->
      <el-tab-pane label="License" name="License">
        <license-key-settings-info-form :tab-name="tabName" :list1="list" />
      </el-tab-pane>
      <!-- ---License--- -->

    </el-tabs>
  </div>
</template>

<script>
import globalAccountInfoTable from './components/globalAccount/InfoTable'
import globalAccountDialogForm from './components/globalAccount/DialogForm'
import monitoringSettingsInfoForm from './components/monitoringSettings/InfoForm'
import licenseKeySettingsInfoForm from './components/license/InfoForm'

export default {
  name: 'Settings',
  components: {
    globalAccountInfoTable,
    globalAccountDialogForm,
    monitoringSettingsInfoForm,
    licenseKeySettingsInfoForm
  },
  data() {
    return {
      tabName: 'globalAccount',
      list: [],
      dialog: {
        visible: false,
        type: 'create',
        formData: {
          id: null,
          user: null,
          password: null,
          account_type: null,
          last_update_date: null
        }
      }
    }
  }
}
</script>

```