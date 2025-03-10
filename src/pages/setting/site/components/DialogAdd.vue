<template>
  <t-dialog v-model:visible="formVisible" header="添加" :width="650" placement="center" :footer="false">
    <template #body>
      <div class="dialog-container-padding">
        <t-form
          :data="formData"
          :rules="rulesSingle"
          :label-width="60"
          @submit="onSubmit"
        >
          <t-form-item label="名称" name="name">
            <t-input v-model="formData.name" placeholder="请输入内容" />
          </t-form-item>
          <t-form-item label="类型" name="type">
            <t-radio-group v-model="formData.type">
              <t-radio :value="0">cms[xml]</t-radio>
              <t-radio :value="1">cms[json]</t-radio>
              <t-radio :value="2">drpy[js0]</t-radio>
              <t-radio :value="6">hipy</t-radio>
              <!-- <t-radio :value="3">drpy[js1]</t-radio> -->
              <t-radio :value="3">app[v3]</t-radio>
              <t-radio :value="4">app[v1]</t-radio>
              <!-- <t-radio :value="5">爬虫</t-radio> -->
            </t-radio-group>
          </t-form-item>
          <t-form-item label="接口" name="api">
            <t-input v-if="formData.type !==5" v-model="formData.api" placeholder="请输入内容" />
            <t-textarea v-else v-model="formData.api" placeholder="请输入内容" />
          </t-form-item>
          <t-form-item label="搜索" name="search">
            <t-radio-group v-model="formData.search">
              <t-radio :value="0">关闭</t-radio>
              <t-radio :value="1">聚合搜索</t-radio>
              <t-radio :value="2">本站搜索</t-radio>
            </t-radio-group>
          </t-form-item>
          <!-- <t-form-item label="筛选" name="filter">
            <t-switch v-model="formData.filter" :label="['开', '关']"/>
          </t-form-item> -->
          <!-- <t-form-item label="下载" name="download">
            <t-input v-model="formData.download" placeholder="请输入内容" />
          </t-form-item> -->
          <t-form-item label="解析" name="playUrl">
            <t-input v-model="formData.playUrl" placeholder="请输入内容" />
          </t-form-item>
          <t-form-item label="扩展" name="ext">
            <t-input v-model="formData.ext" placeholder="请输入内容" />
          </t-form-item>
          <t-form-item label="分组" name="type">
            <t-select
              v-model="formData.group"
              creatable
              filterable
              placeholder="请选择分组"
              @create="createOptions"
            >
              <t-option
                v-for="item in siteGroup"
                :key="item.value"
                :value="item.value"
                :label="item.label"
                class="select-options"
              />
            </t-select>
          </t-form-item>
          <t-form-item label="类别" name="categories">
            <t-input v-model="formData.categories" placeholder="请输入内容,逗号分隔" />
          </t-form-item>
          <div class="optios">
            <t-form-item style="float: right">
              <t-button variant="outline" @click="onClickCloseBtn">取消</t-button>
              <t-button theme="primary" type="submit">确定</t-button>
            </t-form-item>
          </div>
        </t-form>
      </div>
    </template>
  </t-dialog>
</template>

<script setup>
import { MessagePlugin } from 'tdesign-vue-next';
import getUuid from 'uuid-by-string';
import { ref, reactive, watch } from 'vue';

import { sites } from '@/lib/dexie';
import zy from '@/lib/utils/tools';

const props = defineProps({
  visible: {
    type: Boolean,
    default: false,
  },
  data: {
    type: Object,
    default: () => {
      return {};
    },
  },
  group: {
    type: Object,
    default: () => {
      return {};
    },
  },
});
const siteData = ref(props.data);
const siteGroup = ref(props.group);

const formVisible = ref(false);
// key, name, api, download, playUrl, isActive, group;
const formData = reactive({
  key: '',
  name: '',
  api: '',
  download: '',
  playUrl: '',
  group: '',
  search: 2,
  status: true,
  isActive: true,
  type: 1,
});
const onSubmit = () => {
    addSite();
    formVisible.value = false;
};
const onClickCloseBtn = () => {
  formVisible.value = false;
};
const emit = defineEmits(['update:visible', 'refreshTableData']);
watch(
  () => formVisible.value,
  (val) => {
    emit('update:visible', val);
  },
);
watch(
  () => props.visible,
  (val) => {
    formVisible.value = val;
    if (!val) emit('refreshTableData');
  },
);
watch(
  () => props.data,
  (val) => {
    siteData.value = val;
  },
);
watch(
  () => props.group,
  (val) => {
    siteGroup.value = val;
    console.log(`[siteGroup] value:${val.map((item) => item.value).join(',')}`);
  },
);

const rulesSingle = {
  name: [{ required: true, message: '请输入源站名', type: 'error' }],
  api: [{ required: true, message: '请输入接口', type: 'error' }],
  type: [{ required: true, message: '请选择类型', type: 'error' }],
  search: [{ required: true, message: '请选择搜索', type: 'error' }],
  filter: [{ required: true, message: '请选择筛选', type: 'error' }],
};

const addSite = () => {
  if (formData.key) {
    if (!checkSiteKey()) return;
  } else {
    formData.key = getUuid(formData.name, 5);
  }
  if (!formData.group) formData.group = '默认';
  sites
    .add(JSON.parse(JSON.stringify(formData)))
    .then((res) => {
      MessagePlugin.success('添加成功');
      if (res) emit('refreshTableData');
    })
    .catch((error) => {
      MessagePlugin.error(`添加失败: ${error}`);
    })
    .finally(() => {
      formData.key = null;
    });
};
const checkSiteKey = () => {
  for (const s of siteData.value) {
    if (s.key === formData.key) {
      MessagePlugin.warning(`源站标识: ${s.key} 已存在, 请勿重复填写.`);
      return false;
    }
  }
  return true;
};

const createOptions = (val) => {
  const targetIndex = siteGroup.value.findIndex((obj) => obj.label === val);
  if (targetIndex === -1) siteGroup.value.push({ value: val, label: val });
};
</script>

<style lang="less" scoped>
.input-item,
:deep(.t-upload__dragger) {
  width: calc(532px - var(--td-size-1));
}
</style>
