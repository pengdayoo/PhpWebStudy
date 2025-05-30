<template>
  <Conf
    ref="conf"
    :type-flag="'nginx'"
    :default-file="defaultFile"
    :file="file"
    :file-ext="'conf'"
    :show-commond="true"
    @on-type-change="onTypeChange"
  >
    <template #common>
      <Common :setting="commonSetting" />
    </template>
  </Conf>
</template>

<script lang="ts" setup>
  import { computed, ref, watch, Ref, reactive } from 'vue'
  import Conf from '@/components/Conf/index.vue'
  import Common from '@/components/Conf/common.vue'
  import type { CommonSetItem } from '@/components/Conf/setup'
  import { I18nT } from '@lang/index'
  import { debounce } from 'lodash'
  import { uuid } from '@shared/utils'

  const { join } = require('path')

  const conf = ref()
  const commonSetting: Ref<CommonSetItem[]> = ref([])
  const file = computed(() => {
    return join(global.Server.NginxDir, 'common/conf/nginx.conf')
  })
  const defaultFile = computed(() => {
    return join(global.Server.NginxDir, 'common/conf/nginx.conf.default')
  })

  const names: CommonSetItem[] = [
    {
      name: 'keepalive_timeout',
      value: '60',
      enable: true,
      tips() {
        return I18nT('apache.Timeout')
      }
    },
    {
      name: 'gzip',
      value: 'off',
      enable: true,
      options: [
        {
          value: 'off',
          label: 'off'
        },
        {
          value: 'on',
          label: 'on'
        }
      ],
      tips() {
        return I18nT('nginx.gzip')
      }
    },
    {
      name: 'gzip_min_length',
      value: '1k',
      enable: true,
      tips() {
        return I18nT('nginx.gzip_min_length')
      }
    },
    {
      name: 'gzip_comp_level',
      value: '2',
      enable: true,
      tips() {
        return I18nT('nginx.gzip_comp_level')
      }
    },
    {
      name: 'client_max_body_size',
      value: '50m',
      enable: true,
      tips() {
        return I18nT('nginx.client_max_body_size')
      }
    },
    {
      name: 'server_names_hash_bucket_size',
      value: '128',
      enable: true,
      tips() {
        return I18nT('nginx.server_names_hash_bucket_size')
      }
    },
    {
      name: 'server_names_hash_max_size',
      value: '512',
      enable: true,
      tips() {
        return I18nT('nginx.server_names_hash_max_size')
      }
    },
    {
      name: 'client_header_buffer_size',
      value: '32k',
      enable: true,
      tips() {
        return I18nT('nginx.client_header_buffer_size')
      }
    },
    {
      name: 'client_body_buffer_size',
      value: '32k',
      enable: true,
      tips() {
        return I18nT('nginx.client_body_buffer_size')
      }
    }
  ]
  let editConfig = ''
  let watcher: any

  const onSettingUpdate = () => {
    let config = editConfig.replace(/\r\n/gm, '\n')
    const list = ['    #PhpWebStudy-Conf-Common-Begin#']
    commonSetting.value.forEach((item) => {
      const regex = new RegExp(`^[\\s\\n#]?([\\s#]*?)${item.name}(.*?)([^\\n])(\\n|$)`, 'gm')
      config = config.replace(regex, `\n\n`)
      if (item.enable) {
        list.push(`    ${item.name} ${item.value};`)
      }
    })
    list.push('    #PhpWebStudy-Conf-Common-END#')
    config = config
      .replace(
        /([\s\n]?[^\n]*)#PhpWebStudy-Conf-Common-Begin#([\s\S]*?)#PhpWebStudy-Conf-Common-END#/g,
        ''
      )
      .replace(/\n+/g, '\n')
      .trim()
    config = config.replace(/http(.*?)\{(.*?)\n/g, `http {\n${list.join('\n')}\n`)
    conf.value.setEditValue(config)
    editConfig = config
  }

  const getCommonSetting = () => {
    if (watcher) {
      watcher()
    }
    let config = editConfig.replace(/\r\n/gm, '\n')
    const arr = [...names].map((item) => {
      const regex = new RegExp(
        `^[\\s\\n]?((?!#)([\\s]*?))${item.name}(.*?)([^\\n])(\\n|$)`,
        'gm'
      )
      const matchs =
        config.match(regex)?.map((s) => {
          const sarr = s
            .trim()
            .split(' ')
            .filter((s) => !!s.trim())
          const k = sarr.shift()
          const v = sarr.join(' ').replace(';', '')
          return {
            k,
            v
          }
        }) ?? []
      console.log('getCommonSetting: ', matchs, item.name)
      const find = matchs?.find((m) => m.k === item.name)
      item.enable = !!find
      item.value = find?.v ?? item.value
      item.key = uuid()
      return item
    })
    commonSetting.value = reactive(arr) as any
    watcher = watch(commonSetting, debounce(onSettingUpdate, 500), {
      deep: true
    })
  }

  const onTypeChange = (type: 'default' | 'common', config: string) => {
    console.log('onTypeChange: ', type, config)
    if (editConfig !== config) {
      editConfig = config
      getCommonSetting()
    }
  }
</script>
