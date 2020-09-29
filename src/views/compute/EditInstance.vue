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
    <a-form
      :form="form"
      :loading="loading"
      @submit="handleSubmit"
      layout="vertical"
      class="form">
      <a-form-item>
        <span slot="label">
          {{ $t('label.name') }}
          <a-tooltip>
            <a-icon type="info-circle" style="color: rgba(0,0,0,.45)" />
          </a-tooltip>
        </span>
        <span>
          <a-input
            v-decorator="['name']" />
        </span>
      </a-form-item>
      <a-form-item>
        <span slot="label">
          {{ $t('label.displayname') }}
          <a-tooltip>
            <a-icon type="info-circle" style="color: rgba(0,0,0,.45)" />
          </a-tooltip>
        </span>
        <span>
          <a-input
            v-decorator="['displayname']" />
        </span>
      </a-form-item>
      <a-form-item>
        <span slot="label">
          {{ $t('label.ostypeid') }}
          <a-tooltip>
            <a-icon type="info-circle" style="color: rgba(0,0,0,.45)" />
          </a-tooltip>
        </span>
        <span>
          <a-select
          v-decorator="['ostypeid']"
          :loading="lists.osType.loading">
          <a-select-option v-for="ostype in lists.osType.opts" :key="ostype.id">
            {{ ostype.description }}
          </a-select-option>
        </a-select>
        </span>
      </a-form-item>
      <a-form-item>
        <span slot="label">
          {{ $t('label.isdynamicallyscalable') }}
          <a-tooltip>
            <a-icon type="info-circle" style="color: rgba(0,0,0,.45)" />
          </a-tooltip>
        </span>
        <span>
          <a-switch
            v-decorator="['isdynamicallyscalable']"></a-switch>
        </span>
      </a-form-item>
      <a-form-item>
        <span slot="label">
          {{ $t('label.haenable') }}
          <a-tooltip>
            <a-icon type="info-circle" style="color: rgba(0,0,0,.45)" />
          </a-tooltip>
        </span>
        <span>
          <a-switch
            v-decorator="['haenable']"></a-switch>
        </span>
      </a-form-item>
      <a-form-item
        :validate-status="error"
        :help="errorMsg">
        <span slot="label">
          {{ $t('label.group') }}
          <a-tooltip>
            <a-icon type="info-circle" style="color: rgba(0,0,0,.45)" />
          </a-tooltip>
        </span>
        <span>
          <a-input
            v-if="activeNewItem"
            v-model="newItem"
            ref="newItem"
            @keyup="resetError">
            <a-button
              slot="addonAfter"
              class="btn-addon"
              icon="check"
              @click="addGroup"></a-button>
            <a-button
              slot="addonAfter"
              class="btn-addon"
              icon="close"
              @click="closeItem"></a-button>
          </a-input>
          <a-select
            v-else
            v-decorator="['group']"
            :loading="lists.group.loading">
            <div slot="dropdownRender" slot-scope="menu">
              <v-nodes :vnodes="menu" />
              <a-divider style="margin: 0;" />
              <a-button
                class="btn-add-item"
                type="default"
                @mousedown="e => e.preventDefault()"
                @click="addItem"
                icon="plus"
              >
                {{ $t('label.add.item') }}
              </a-button>
            </div>
            <a-select-option v-for="ostype in lists.group.opts" :key="ostype.id">
              {{ ostype.name }}
            </a-select-option>
          </a-select>
        </span>
      </a-form-item>
    </a-form>
  </div>
</template>

<script>
import { api } from '@/api'

export default {
  name: 'EditInstance',
  components: {
    VNodes: {
      functional: true,
      render: (h, ctx) => ctx.props.vnodes
    }
  },
  props: {
    resource: {
      type: Object,
      required: true
    },
    loading: {
      type: Boolean,
      default: () => false
    },
    action: {
      type: Object,
      default: () => {}
    }
  },
  data () {
    return {
      selectedGroup: '',
      newItem: '',
      activeNewItem: false,
      error: '',
      errorMsg: '',
      lists: {
        osType: {
          loading: false,
          opts: []
        },
        group: {
          loading: false,
          opts: []
        }
      }
    }
  },
  beforeCreate () {
    this.form = this.$form.createForm(this)
  },
  mounted () {
    this.fetchData()
  },
  methods: {
    fetchData () {
      this.fetchOsType()
      this.fetchInstanceGroup()
      this.fillEditFormFieldValues()
    },
    fetchOsType () {
      this.lists.osType.loading = true
      api('listOsTypes', { listAll: true }).then(json => {
        this.lists.osType.opts = json.listostypesresponse.ostype || []
      }).catch(error => {
        this.$notifyError(error)
      }).finally(() => {
        this.lists.osType.loading = false
      })
    },
    fetchInstanceGroup () {
      this.lists.group.loading = true
      api('listInstanceGroups', { listAll: true }).then(json => {
        this.lists.group.opts = json.listinstancegroupsresponse.instancegroup || []
      }).catch(error => {
        this.$notifyError(error)
      }).finally(() => {
        this.lists.group.loading = false
      })
    },
    fillEditFormFieldValues () {
      const form = this.form
      const fields = this.form.getFieldsValue()
      Object.keys(fields).map(field => {
        let fieldValue = null
        const fieldName = field
        fieldValue = this.resource[fieldName] ? this.resource[fieldName] : null
        if (fieldValue) {
          form.getFieldDecorator(fieldName, { initialValue: fieldValue })
          form.setFieldsValue({ [fieldName]: fieldValue })
        }
      })
    },
    handleSubmit (e) {},
    addItem () {
      this.activeNewItem = true
      setTimeout(() => {
        this.$refs.newItem.focus()
      })
    },
    addGroup () {
      if (!this.validateItem()) {
        return
      }
      const randomUuid = this.randomUuid()
      const group = {
        id: randomUuid,
        name: this.newItem,
        type: 'new-group'
      }
      this.form.getFieldDecorator('group', { initialValue: randomUuid })
      this.lists.group.opts.push(group)
      this.closeItem()
    },
    closeItem () {
      this.activeNewItem = false
      this.newItem = ''
      this.resetError()
    },
    validateItem () {
      this.resetError()
      if (!this.newItem || this.newItem.length === 0) {
        this.error = 'error'
        this.errorMsg = `${this.$t('message.error.required.input')}`
        return false
      }
      const groupIdx = this.lists.group.opts.findIndex(group => group.name === this.newItem)
      if (groupIdx > -1) {
        this.error = 'error'
        this.errorMsg = `${this.$t('label.group')} ${this.newItem} ${this.$t('label.already.exists')}`
        return false
      }
      return true
    },
    resetError () {
      this.error = ''
      this.errorMsg = ''
    },
    randomUuid () {
      return Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15)
    }
  }
}
</script>

<style scoped lang="less">
.form {
  width: 90vw;
  @media (min-width: 700px) {
    width: 50vw;
  }
}
.btn-add-item {
  display: block;
  width: 100%;
}
/deep/.ant-input-group-addon {
  padding: 0;

  .btn-addon {
    border: 0;
    height: auto;
    box-shadow: none;
    background: transparent;
  }
}
</style>
