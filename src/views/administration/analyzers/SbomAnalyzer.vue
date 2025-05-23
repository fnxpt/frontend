<template>
  <b-card no-body :header="header">
    <b-card-body>
      <c-switch
        :disabled="!this.scannerEnabled && (!this.baseUrl || !this.apitoken)"
        id="scannerEnabled"
        color="primary"
        v-model="scannerEnabled"
        label
        v-bind="labelIcon"
      />
      {{ $t('admin.analyzer_sbom_enable') }}
      <b-validated-input-group-form-input
        id="sbom-baseUrl"
        :label="$t('admin.base_url')"
        input-group-size="mb-3"
        rules="required"
        v-model="baseUrl"
        lazy="true"
      />
      <b-validated-input-group-form-input
        id="sbom-apitoken"
        :label="$t('admin.api_token')"
        input-group-size="mb-3"
        rules="required"
        type="password"
        v-model="apitoken"
        lazy="true"
      />
    </b-card-body>
    <b-card-footer>
      <b-button
        :disabled="!this.baseUrl || !this.apitoken"
        variant="outline-primary"
        class="px-4"
        @click="saveChanges"
      >
        {{ $t('message.update') }}
      </b-button>
    </b-card-footer>
  </b-card>
</template>

<script>
import { Switch as cSwitch } from '@coreui/vue';
import BValidatedInputGroupFormInput from '../../../forms/BValidatedInputGroupFormInput';
import common from '../../../shared/common';
import configPropertyMixin from '../mixins/configPropertyMixin';
export default {
  mixins: [configPropertyMixin],
  props: {
    header: String,
  },
  components: {
    cSwitch,
    BValidatedInputGroupFormInput,
  },
  data() {
    return {
      scannerEnabled: false,
      apitoken: '',
      baseUrl: '',
      ignoreUnfixed: false,
      scanLibrary: true,
      scanOs: true,
    };
  },
  methods: {
    saveChanges: function () {
      this.updateConfigProperties([
        {
          groupName: 'scanner',
          propertyName: 'sbom.enabled',
          propertyValue: this.scannerEnabled,
        },
        {
          groupName: 'scanner',
          propertyName: 'sbom.api.token',
          propertyValue: this.apitoken,
        },
        {
          groupName: 'scanner',
          propertyName: 'sbom.base.url',
          propertyValue: this.baseUrl,
        },
      ]);
    },
  },
  created() {
    this.axios.get(this.configUrl).then((response) => {
      let configItems = response.data.filter(function (item) {
        return item.groupName === 'scanner';
      });
      for (let i = 0; i < configItems.length; i++) {
        let item = configItems[i];
        switch (item.propertyName) {
          case 'sbom.enabled':
            this.scannerEnabled = common.toBoolean(item.propertyValue);
            break;
          case 'sbom.api.token':
            this.apitoken = item.propertyValue;
            break;
          case 'sbom.base.url':
            this.baseUrl = item.propertyValue;
            break;
        }
      }
    });
  },
};
</script>
