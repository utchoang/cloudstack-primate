// Licensed to the Apache Software Foundation (ASF) under one
// or more contributor license agreements.  See the NOTICE file
// distributed with this work for additional information
// regarding copyright ownership.  The ASF licenses this file
// to you under the Apache License, Version 2.0 (the
// "License"); you may not use this file except in compliance
// with the License.  You may obtain a copy of the License at
//
//   http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing,
// software distributed under the License is distributed on an
// "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
// KIND, either express or implied.  See the License for the
// specific language governing permissions and limitations
// under the License.

<template>
  <div class="form-layout">
    <a-spin :spinning="loading">
      <a-form
        :form="form"
        @submit="handleSubmit"
        layout="vertical">

        <a-form-item :label="$t('ispublic')" v-show="this.isAdmin()">
          <a-switch v-decorator="['ispublic', {initialValue: this.isPublic}]" :defaultChecked="this.offeringIsPublic" @change="val => { this.offeringIsPublic = val }" />
        </a-form-item>

        <a-form-item :label="$t('domainid')" v-if="!this.offeringIsPublic">
          <a-select
            mode="multiple"
            v-decorator="['domainid', {
              rules: [
                {
                  required: true,
                  message: 'Please select option'
                }
              ]
            }]"
            showSearch
            optionFilterProp="children"
            :filterOption="(input, option) => {
              return option.componentOptions.children[0].text.toLowerCase().indexOf(input.toLowerCase()) >= 0
            }"
            :loading="domainLoading"
            :placeholder="this.$t('label.domain')">
            <a-select-option v-for="(opt, optIndex) in this.domains" :key="optIndex">
              {{ opt.name || opt.description }}
            </a-select-option>
          </a-select>
        </a-form-item>

        <a-form-item :label="$t('zoneid')">
          <a-select
            id="zone-selection"
            mode="multiple"
            v-decorator="['zoneid', {
              rules: [
                {
                  validator: (rule, value, callback) => {
                    if (value && value.length > 1 && value.indexOf(0) !== -1) {
                      callback('All Zones cannot be combined with any other zone')
                    }
                    callback()
                  }
                }
              ]
            }]"
            showSearch
            optionFilterProp="children"
            :filterOption="(input, option) => {
              return option.componentOptions.children[0].text.toLowerCase().indexOf(input.toLowerCase()) >= 0
            }"
            :loading="zoneLoading"
            :placeholder="this.$t('zone')">
            <a-select-option v-for="(opt, optIndex) in this.zones" :key="optIndex">
              {{ opt.name || opt.description }}
            </a-select-option>
          </a-select>
        </a-form-item>

        <div :span="24" class="action-button">
          <a-button @click="closeAction">{{ this.$t('Cancel') }}</a-button>
          <a-button :loading="loading" type="primary" @click="handleSubmit">{{ this.$t('OK') }}</a-button>
        </div>

      </a-form>
    </a-spin>
  </div>
</template>

<script>
import { api } from '@/api'

export default {
  name: 'UpdateOfferingAccess',
  props: {
    resource: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      offeringType: '',
      selectedDomains: [],
      selectedZones: [],
      offeringIsPublic: false,
      domains: [],
      domainLoading: false,
      zones: [],
      zoneLoading: false,
      loading: false
    }
  },
  beforeCreate () {
    this.form = this.$form.createForm(this)
  },
  created () {
    this.zones = [
      {
        id: 'all',
        name: this.$t('label.all.zone')
      }
    ]
  },
  mounted () {
    switch (this.$route.meta.name) {
      case 'computeoffering':
        this.offeringType = 'ServiceOffering'
        break
      case 'diskoffering':
        this.offeringType = 'DiskOffering'
        break
      case 'networkoffering':
        this.offeringType = 'NetworkOffering'
        break
      case 'vpcoffering':
        this.offeringType = 'VPCOffering'
        break
      default:
        this.offeringType = this.$route.meta.name
    }
    this.fetchData()
  },
  methods: {
    fetchData () {
      this.fetchDomainData()
      this.fetchZoneData()
    },
    isAdmin () {
      return ['Admin'].includes(this.$store.getters.userInfo.roletype)
    },
    fetchDomainData () {
      const params = {}
      params.listAll = true
      params.details = 'min'
      this.domainLoading = true
      api('listDomains', params).then(json => {
        const listDomains = json.listdomainsresponse.domain
        this.domains = this.domains.concat(listDomains)
      }).finally(() => {
        this.domainLoading = false
        this.updateDomainSelection()
      })
    },
    fetchZoneData () {
      const params = {}
      params.listAll = true
      this.zoneLoading = true
      api('listZones', params).then(json => {
        const listZones = json.listzonesresponse.zone
        this.zones = this.zones.concat(listZones)
      }).finally(() => {
        this.zoneLoading = false
        this.updateZoneSelection()
      })
    },
    updateDomainSelection () {
      var offeringDomainIds = this.resource.domainid
      this.selectedDomains = []
      if (offeringDomainIds) {
        this.offeringIsPublic = false
        offeringDomainIds = offeringDomainIds.indexOf(',') !== -1 ? offeringDomainIds.split(',') : [offeringDomainIds]
        for (var i = 0; i < offeringDomainIds.length; i++) {
          for (var j = 0; j < this.domains.length; j++) {
            if (offeringDomainIds[i] === this.domains[j].id) {
              this.selectedDomains = this.selectedDomains.concat(j)
            }
          }
        }
      } else {
        if (this.isAdmin()) {
          this.offeringIsPublic = true
        }
      }
      this.form.setFieldsValue({
        domainid: this.selectedDomains
      })
    },
    updateZoneSelection () {
      var offeringZoneIds = this.resource.zoneid
      this.selectedZones = []
      if (offeringZoneIds) {
        offeringZoneIds = offeringZoneIds.indexOf(',') !== -1 ? offeringZoneIds.split(',') : [offeringZoneIds]
        for (var i = 0; i < offeringZoneIds.length; i++) {
          for (var j = 0; j < this.zones.length; j++) {
            if (offeringZoneIds[i] === this.zones[j].id) {
              this.selectedZones = this.selectedZones.concat(j)
            }
          }
        }
      }
      this.form.setFieldsValue({
        zoneid: this.selectedZones
      })
    },
    handleSubmit (e) {
      e.preventDefault()
      this.form.validateFields((err, values) => {
        if (err) {
          return
        }

        const params = {}
        params.id = this.resource.id
        var ispublic = values.ispublic
        if (ispublic === true) {
          params.domainid = 'public'
        } else {
          var domainIndexes = values.domainid
          var domainId = 'public'
          if (domainIndexes && domainIndexes.length > 0) {
            var domainIds = []
            for (var i = 0; i < domainIndexes.length; i++) {
              domainIds = domainIds.concat(this.domains[domainIndexes[i]].id)
            }
            domainId = domainIds.join(',')
          }
          params.domainid = domainId
        }
        var zoneIndexes = values.zoneid
        var zoneId = 'all'
        if (zoneIndexes && zoneIndexes.length > 0) {
          var zoneIds = []
          for (var j = 0; j < zoneIndexes.length; j++) {
            zoneIds = zoneIds.concat(this.zones[zoneIndexes[j]].id)
          }
          zoneId = zoneIds.join(',')
        }
        params.zoneid = zoneId

        this.loading = true
        api('update' + this.offeringType, params).then(json => {
          this.$emit('refresh-data')
          this.$notification.success({
            message: this.$t('label.action.update.offering.access'),
            description: this.$t('label.action.update.offering.access')
          })
        }).catch(error => {
          this.$notification.error({
            message: 'Request Failed',
            description: (error.response && error.response.headers && error.response.headers['x-description']) || error.message
          })
        }).finally(() => {
          this.loading = false
          this.closeAction()
        })
      })
    },
    closeAction () {
      this.$emit('close-action')
    }
  }
}
</script>

<style scoped lang="scss">
  .form-layout {
    width: 80vw;

    @media (min-width: 600px) {
      width: 25vw;
    }
  }

  .action-button {
    text-align: right;

    button {
      margin-right: 5px;
    }
  }
</style>
