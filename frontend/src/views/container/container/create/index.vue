<template>
    <el-drawer v-model="drawerVisiable" :destroy-on-close="true" :close-on-click-modal="false" size="50%">
        <template #header>
            <DrawerHeader :header="$t('container.createContainer')" :back="handleClose" />
        </template>
        <el-form ref="formRef" label-position="top" v-loading="loading" :model="form" :rules="rules" label-width="80px">
            <el-row type="flex" justify="center">
                <el-col :span="22">
                    <el-form-item :label="$t('container.name')" prop="name">
                        <el-input clearable v-model.trim="form.name" />
                    </el-form-item>
                    <el-form-item :label="$t('container.image')" prop="image">
                        <el-select style="width: 100%" allow-create filterable v-model="form.image">
                            <el-option
                                v-for="(item, index) of images"
                                :key="index"
                                :value="item.option"
                                :label="item.option"
                            />
                        </el-select>
                    </el-form-item>
                    <el-form-item :label="$t('container.port')">
                        <el-radio-group v-model="form.publishAllPorts" class="ml-4">
                            <el-radio :label="false">{{ $t('container.exposePort') }}</el-radio>
                            <el-radio :label="true">{{ $t('container.exposeAll') }}</el-radio>
                        </el-radio-group>
                    </el-form-item>
                    <el-form-item v-if="!form.publishAllPorts">
                        <el-card style="width: 100%">
                            <table style="width: 100%" class="tab-table">
                                <tr v-if="form.exposedPorts.length !== 0">
                                    <th scope="col" width="48%" align="left">
                                        <label>{{ $t('container.serverPort') }}</label>
                                    </th>
                                    <th scope="col" width="48%" align="left">
                                        <label>{{ $t('container.containerPort') }}</label>
                                    </th>
                                    <th align="left"></th>
                                </tr>
                                <tr v-for="(row, index) in form.exposedPorts" :key="index">
                                    <td width="48%">
                                        <el-input-number
                                            :min="0"
                                            :max="65535"
                                            style="width: 100%"
                                            controls-position="right"
                                            v-model.number="row.hostPort"
                                        />
                                    </td>
                                    <td width="48%">
                                        <el-input-number
                                            :min="0"
                                            :max="65535"
                                            style="width: 100%"
                                            controls-position="right"
                                            v-model.number="row.containerPort"
                                        />
                                    </td>
                                    <td>
                                        <el-button link style="font-size: 10px" @click="handlePortsDelete(index)">
                                            {{ $t('commons.button.delete') }}
                                        </el-button>
                                    </td>
                                </tr>
                                <tr>
                                    <td align="left">
                                        <el-button @click="handlePortsAdd()">{{ $t('commons.button.add') }}</el-button>
                                    </td>
                                </tr>
                            </table>
                        </el-card>
                    </el-form-item>
                    <el-form-item :label="$t('container.cmd')" prop="cmdStr">
                        <el-input
                            type="textarea"
                            :placeholder="$t('container.cmdHelper')"
                            :autosize="{ minRows: 2, maxRows: 4 }"
                            v-model="form.cmdStr"
                        />
                    </el-form-item>
                    <el-form-item prop="autoRemove">
                        <el-checkbox v-model="form.autoRemove">{{ $t('container.autoRemove') }}</el-checkbox>
                    </el-form-item>
                    <el-form-item :label="$t('container.cpuQuota')" prop="nanoCPUs">
                        <el-input type="number" style="width: 40%" v-model.number="form.nanoCPUs">
                            <template #append>
                                <el-select v-model="form.cpuUnit" disabled style="width: 85px">
                                    <el-option label="Core" value="Core" />
                                </el-select>
                            </template>
                        </el-input>
                        <span class="input-help">{{ $t('container.limitHelper') }}</span>
                    </el-form-item>
                    <el-form-item :label="$t('container.memoryLimit')" prop="memoryItem">
                        <el-input style="width: 40%" v-model.number="form.memoryItem">
                            <template #append>
                                <el-select v-model="form.memoryUnit" placeholder="Select" style="width: 85px">
                                    <el-option label="KB" value="KB" />
                                    <el-option label="MB" value="MB" />
                                    <el-option label="GB" value="GB" />
                                </el-select>
                            </template>
                        </el-input>
                        <span class="input-help">{{ $t('container.limitHelper') }}</span>
                    </el-form-item>
                    <el-form-item :label="$t('container.mount')">
                        <el-card style="width: 100%">
                            <table style="width: 100%" class="tab-table">
                                <tr v-if="form.volumes.length !== 0">
                                    <th scope="col" width="42%" align="left">
                                        <label>{{ $t('container.serverPath') }}</label>
                                    </th>
                                    <th scope="col" width="12%" align="left">
                                        <label>{{ $t('container.mode') }}</label>
                                    </th>
                                    <th scope="col" width="42%" align="left">
                                        <label>{{ $t('container.containerDir') }}</label>
                                    </th>
                                    <th align="left"></th>
                                </tr>
                                <tr v-for="(row, index) in form.volumes" :key="index">
                                    <td width="42%">
                                        <el-select
                                            style="width: 100%"
                                            allow-create
                                            clearable
                                            :placeholder="$t('commons.msg.inputOrSelect')"
                                            filterable
                                            v-model="row.sourceDir"
                                        >
                                            <el-option
                                                v-for="(item, indexV) of volumes"
                                                :key="indexV"
                                                :value="item.option"
                                                :label="item.option"
                                            />
                                        </el-select>
                                    </td>
                                    <td width="12%">
                                        <el-select style="width: 100%" filterable v-model="row.mode">
                                            <el-option value="rw" :label="$t('container.modeRW')" />
                                            <el-option value="ro" :label="$t('container.modeR')" />
                                        </el-select>
                                    </td>
                                    <td width="42%">
                                        <el-input v-model="row.containerDir" />
                                    </td>
                                    <td>
                                        <el-button link style="font-size: 10px" @click="handleVolumesDelete(index)">
                                            {{ $t('commons.button.delete') }}
                                        </el-button>
                                    </td>
                                </tr>
                                <tr>
                                    <td align="left">
                                        <el-button @click="handleVolumesAdd()">
                                            {{ $t('commons.button.add') }}
                                        </el-button>
                                    </td>
                                </tr>
                            </table>
                        </el-card>
                    </el-form-item>
                    <el-form-item :label="$t('container.tag')" prop="labelsStr">
                        <el-input
                            type="textarea"
                            :placeholder="$t('container.tagHelper')"
                            :autosize="{ minRows: 2, maxRows: 4 }"
                            v-model="form.labelsStr"
                        />
                    </el-form-item>
                    <el-form-item :label="$t('container.env')" prop="envStr">
                        <el-input
                            type="textarea"
                            :placeholder="$t('container.tagHelper')"
                            :autosize="{ minRows: 2, maxRows: 4 }"
                            v-model="form.envStr"
                        />
                    </el-form-item>
                    <el-form-item :label="$t('container.restartPolicy')" prop="restartPolicy">
                        <el-radio-group v-model="form.restartPolicy">
                            <el-radio label="unless-stopped">{{ $t('container.unlessStopped') }}</el-radio>
                            <el-radio label="on-failure">{{ $t('container.onFailure') }}</el-radio>
                            <el-radio label="no">{{ $t('container.no') }}</el-radio>
                        </el-radio-group>
                    </el-form-item>
                </el-col>
            </el-row>
        </el-form>
        <template #footer>
            <span class="dialog-footer">
                <el-button :disabled="loading" @click="drawerVisiable = false">
                    {{ $t('commons.button.cancel') }}
                </el-button>
                <el-button :disabled="loading" type="primary" @click="onSubmit(formRef)">
                    {{ $t('commons.button.confirm') }}
                </el-button>
            </span>
        </template>
    </el-drawer>
</template>

<script lang="ts" setup>
import { reactive, ref } from 'vue';
import { Rules } from '@/global/form-rules';
import i18n from '@/lang';
import { ElForm } from 'element-plus';
import DrawerHeader from '@/components/drawer-header/index.vue';
import { listImage, listVolume, createContainer } from '@/api/modules/container';
import { Container } from '@/api/interface/container';
import { MsgError, MsgSuccess } from '@/utils/message';

const loading = ref(false);

const drawerVisiable = ref(false);
const form = reactive({
    name: '',
    image: '',
    cmdStr: '',
    cmd: [] as Array<string>,
    publishAllPorts: false,
    exposedPorts: [] as Array<Container.Port>,
    nanoCPUs: 0,
    memory: 0,
    memoryItem: 0,
    memoryUnit: 'MB',
    cpuUnit: 'Core',
    volumes: [] as Array<Container.Volume>,
    autoRemove: false,
    labels: [] as Array<string>,
    labelsStr: '',
    env: [] as Array<string>,
    envStr: '',
    restartPolicy: '',
});
const images = ref();
const volumes = ref();

const acceptParams = (): void => {
    handlReset();
    drawerVisiable.value = true;
    loadImageOptions();
    loadVolumeOptions();
};

const handlReset = () => {
    form.name = '';
    form.image = '';
    form.cmdStr = '';
    form.cmd = [];
    form.publishAllPorts = false;
    form.exposedPorts = [];
    form.nanoCPUs = 0;
    form.memory = 0;
    form.memoryItem = 0;
    form.memoryUnit = 'MB';
    form.cpuUnit = 'Core';
    form.volumes = [];
    form.autoRemove = false;
    form.labels = [];
    form.labelsStr = '';
    form.env = [];
    form.envStr = '';
    form.restartPolicy = 'no';
};

const handleClose = () => {
    drawerVisiable.value = false;
};

const emit = defineEmits<{ (e: 'search'): void }>();

const rules = reactive({
    name: [Rules.requiredInput, Rules.name],
    image: [Rules.requiredSelect],
    nanoCPUs: [Rules.number],
    memoryItem: [Rules.number],
});

type FormInstance = InstanceType<typeof ElForm>;
const formRef = ref<FormInstance>();

const handlePortsAdd = () => {
    let item = {
        containerPort: null,
        hostPort: null,
    };
    form.exposedPorts.push(item);
};
const handlePortsDelete = (index: number) => {
    form.exposedPorts.splice(index, 1);
};

const handleVolumesAdd = () => {
    let item = {
        sourceDir: '',
        containerDir: '',
        mode: 'rw',
    };
    form.volumes.push(item);
};
const handleVolumesDelete = (index: number) => {
    form.volumes.splice(index, 1);
};

const loadImageOptions = async () => {
    const res = await listImage();
    images.value = res.data;
};
const loadVolumeOptions = async () => {
    const res = await listVolume();
    volumes.value = res.data;
};
const onSubmit = async (formEl: FormInstance | undefined) => {
    if (form.volumes.length !== 0) {
        for (const item of form.volumes) {
            if (!item.containerDir || !item.sourceDir) {
                MsgError(i18n.global.t('container.volumeHelper'));
                return;
            }
        }
    }
    if (!formEl) return;
    formEl.validate(async (valid) => {
        if (!valid) return;
        if (form.envStr.length !== 0) {
            form.env = form.envStr.split('\n');
        }
        if (form.labelsStr.length !== 0) {
            form.labels = form.labelsStr.split('\n');
        }
        if (form.cmdStr.length !== 0) {
            form.cmd = form.cmdStr.split('\n');
        }
        switch (form.memoryUnit) {
            case 'KB':
                form.memory = form.memoryItem * 1024;
                break;
            case 'MB':
                form.memory = form.memoryItem * 1024 * 1024;
                break;
            case 'GB':
                form.memory = form.memoryItem * 1024 * 1024 * 1024;
                break;
        }
        loading.value = true;
        await createContainer(form)
            .then(() => {
                loading.value = false;
                MsgSuccess(i18n.global.t('commons.msg.operationSuccess'));
                emit('search');
                drawerVisiable.value = false;
            })
            .catch(() => {
                loading.value = false;
            });
    });
};

defineExpose({
    acceptParams,
});
</script>
