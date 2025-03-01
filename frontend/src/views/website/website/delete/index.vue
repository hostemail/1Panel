<template>
    <el-dialog
        v-model="open"
        :close-on-click-modal="false"
        :title="$t('website.delete')"
        width="30%"
        :before-close="handleClose"
    >
        <div :key="key" v-loading="loading">
            <el-form ref="deleteForm" label-position="left">
                <el-form-item>
                    <el-checkbox v-model="deleteReq.forceDelete" :label="$t('website.forceDelete')" />
                    <span class="input-help">
                        {{ $t('website.forceDeleteHelper') }}
                    </span>
                </el-form-item>
                <el-form-item v-if="type === 'deployment'">
                    <el-checkbox v-model="deleteReq.deleteApp" :label="$t('website.deleteApp')" />
                    <span class="input-help">
                        {{ $t('website.deleteAppHelper') }}
                    </span>
                </el-form-item>
                <el-form-item>
                    <el-checkbox v-model="deleteReq.deleteBackup" :label="$t('website.deleteBackup')" />
                    <span class="input-help">
                        {{ $t('website.deleteBackupHelper') }}
                    </span>
                </el-form-item>
                <el-form-item>
                    <span v-html="deleteHelper"></span>
                    <el-input v-model="deleteInfo" :placeholder="websiteName" />
                </el-form-item>
            </el-form>
        </div>
        <template #footer>
            <span class="dialog-footer">
                <el-button @click="handleClose" :disabled="loading">{{ $t('commons.button.cancel') }}</el-button>
                <el-button type="primary" @click="submit()" :disabled="loading || deleteInfo != websiteName">
                    {{ $t('commons.button.confirm') }}
                </el-button>
            </span>
        </template>
    </el-dialog>
</template>

<script lang="ts" setup>
import { DeleteWebsite } from '@/api/modules/website';
import i18n from '@/lang';
import { FormInstance } from 'element-plus';
import { ref } from 'vue';
import { Website } from '@/api/interface/website';
import { MsgSuccess } from '@/utils/message';

let key = 1;
let open = ref(false);
let loading = ref(false);
let deleteReq = ref({
    id: 0,
    deleteApp: false,
    deleteBackup: false,
    forceDelete: false,
});
let type = ref('');
const em = defineEmits(['close']);
const deleteForm = ref<FormInstance>();
let deleteInfo = ref('');
let websiteName = ref('');
let deleteHelper = ref('');

const handleClose = () => {
    open.value = false;
    em('close', false);
};

const acceptParams = async (website: Website.Website) => {
    deleteReq.value = {
        id: 0,
        deleteApp: false,
        deleteBackup: false,
        forceDelete: false,
    };
    deleteInfo.value = '';
    deleteReq.value.id = website.id;
    websiteName.value = website.primaryDomain;
    deleteHelper.value = i18n.global.t('website.deleteConfirmHelper', [website.primaryDomain]);
    type.value = website.type;
    open.value = true;
};

const submit = () => {
    loading.value = true;
    DeleteWebsite(deleteReq.value)
        .then(() => {
            handleClose();
            MsgSuccess(i18n.global.t('commons.msg.deleteSuccess'));
        })
        .finally(() => {
            loading.value = false;
        });
};

defineExpose({
    acceptParams,
});
</script>
