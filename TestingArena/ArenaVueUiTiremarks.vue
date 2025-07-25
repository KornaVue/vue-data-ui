<script setup>
import { ref, computed, onMounted } from "vue";
import LocalVueUiTiremarks from '../src/components/vue-ui-tiremarks.vue';
import LocalVueDataUi from '../src/components/vue-data-ui.vue';
import Box from "./Box.vue";
import convertArrayToObject from "./convertModel";

const dataset = ref({ percentage: 66.6 })

onMounted(() => {
    setTimeout(() => {
        dataset.value.percentage = 10;
    }, 3000)
})

const isPropsToggled = ref(false);
function toggleProps() {
    isPropsToggled.value = !isPropsToggled.value;
}

const alternateDataset = ref({ percentage: 50 });

const alternateConfig = ref({
    style: {
        chart: {
            backgroundColor: '#FF0000',
            title: {
                text: 'Alternate'
            }
        }
    }
})

function alterDataset() {
    dataset.value.percentage = Math.round(Math.random() * 100);
}

const model = ref([
    { key: 'userOptions.show', def: true, type: 'checkbox'},
    { key: 'userOptions.buttons.pdf', def: true, type: 'checkbox'},
    { key: 'userOptions.buttons.img', def: true, type: 'checkbox'},
    { key: 'userOptions.buttons.fullscreen', def: true, type: 'checkbox'},
    { key: 'userOptions.position', def: 'right', type: 'select', options: ['left', 'right']},
    { key: 'userOptions.showOnChartHover', def: true, type: 'checkbox'},
    { key: 'userOptions.keepStateOnChartLeave', def: true, type: 'checkbox'},
    
    { key: 'style.fontFamily', def: 'inherit', type: 'text'},
    { key: 'style.chart.backgroundColor', def: '#FFFFFF20', type: 'color'},
    { key: 'style.chart.color', def: '#1A1A1A', type: 'color'},
    { key: 'style.chart.animation.use', def: true, type: 'checkbox'},
    { key: 'style.chart.animation.speed', def: 0.5, type: 'number', min: 0, max: 2, step: 0.01},
    { key: 'style.chart.animation.acceleration', def: 1, type: 'number', min: 0, max: 10, step: 0.1},
    { key: 'style.chart.layout.display', def: 'horizontal', type: 'select', options: ['horizontal', 'vertical']},
    { key: 'style.chart.layout.crescendo', def: false, type: 'checkbox'},
    { key: 'style.chart.layout.curved', def: false, type: 'checkbox'},
    { key: 'style.chart.layout.curveAngleX', def: 10, type:'number', min: -360, max: 360},
    { key: 'style.chart.layout.curveAngleY', def: 10, type: 'number', min: -360, max: 360},
    { key: 'style.chart.layout.activeColor', def: '#5f8bee', type: 'color'},
    { key: 'style.chart.layout.inactiveColor', def: '#e1e5e8', type: 'color'},
    { key: 'style.chart.layout.ticks.gradient.show', def: true, type: 'checkbox'},
    { key: 'style.chart.layout.ticks.gradient.shiftHueIntensity', def: 100, type: 'range', min: 0, max: 100},
    { key: 'style.chart.percentage.show', def: true, type: 'checkbox'},
    { key: 'style.chart.percentage.fontSize', def: 16, type: 'range', min: 8, max: 100},
    { key: 'style.chart.percentage.rounding', def: 1, type: 'range', min: 0, max: 12},
    { key: 'style.chart.percentage.bold', def: true, type: 'checkbox'},
    { key: 'style.chart.percentage.useGradientColor', def: true, type: 'checkbox'},
    { key: 'style.chart.percentage.color', def: '#1A1A1A', type: 'color'},
    { key: 'style.chart.percentage.verticalPosition', def: 'bottom', type: 'select', options:['top', 'bottom']},
    { key: 'style.chart.percentage.horizontalPosition', def: 'left', type: 'select', options: ['left', 'right', 'top']},
    { key: 'style.chart.title.text', def: 'Lorem ipsum dolor sic amet', type: 'text'},
    { key: 'style.chart.title.text', def: 'At vero eos et accusamus et iusto odio dignissimos ducimus qui blanditiis', type: 'text'},
    { key: 'style.chart.title.color', def: '#1A1A1A', type: 'color'},
    { key: 'style.chart.title.fontSize', def: 20, type: 'number', min: 8, max: 48},
    { key: 'style.chart.title.bold', def: true, type: 'checkbox'},
    { key: 'style.chart.title.subtitle.text', def: 'At vero eos et accusamus et iusto odio dignissimos ducimus qui blanditiis', type: 'text'},
    { key: 'style.chart.title.subtitle.color', def: '#CCCCCC', type: 'color'},
    { key: 'style.chart.title.subtitle.fontSize', def: 16, type: 'range', min: 8, max: 48},
    { key: 'style.chart.title.subtitle.bold', def: false, type: 'checkbox'},

    { key: 'userOptions.print.scale', def: 2, type: 'number', min: 1, max: 5},
    { key: 'userOptions.print.allowTaint', def: true, type: 'checkbox'},
    { key: 'userOptions.print.useCORS', def: true, type: 'checkbox'},
    { key: 'userOptions.print.backgroundColor', def: '#FFFFFF' }

])

const themeOptions = ref([
    "",
    "hack",
    "zen",
    "concrete",
    "default",
    "celebration",
    "celebrationNight"
])

const currentTheme = ref(themeOptions.value[6])

const config = computed(() => {
    const c = convertArrayToObject(model.value);
    return {
        ...c,
        style: {
            ...c.style,
            chart: {
                ...c.style.chart,
                percentage: {
                    ...c.style.chart.percentage,
                    formatter: ({value}) => {
                        return `f - ${value}`
                    }
                }
            }
        },
        theme: currentTheme.value
    }
});

const step = ref(0)

const local = ref(null)

onMounted(async () => {
    if (local.value) {
        const img = await local.value.getImage()
        console.log(img)
    }
})

</script>

<template>
    <div style="margin: 12px 0; color: white">
        Theme:
        <select v-model="currentTheme" @change="step += 1">
            <option v-for="opt in themeOptions">{{ opt }}</option>
        </select>
    </div>

    <button @click="toggleProps">TOGGLE PROPS: {{ isPropsToggled }}</button>
    <button @click="alterDataset">ALTER DATASET</button>

    <Box comp="VueUiTiremarks" :dataset="dataset">
        <template #title>VueUiTiremarks</template>

        <template #local>
            <LocalVueUiTiremarks :dataset="isPropsToggled ? alternateDataset : dataset" :config="isPropsToggled ? alternateConfig : config" :key="`local_${step}`" ref="local">
                <template #chart-background>
                    <div style="width: 100%; height: 100%; background: radial-gradient(at top left, red, white)"/>
                </template>

                <template #optionPdf>
                    PRINT PDF
                </template>
                <template #watermark="{ isPrinting }">
                    <div v-if="isPrinting" style="font-size: 100px; opacity: 0.1; transform: rotate(-10deg)">
                        WATERMARK
                    </div>
                </template>
                <template #source>
                    <div style="width:100%;font-size:10px;text-align:left">
                        SOURCE: Lorem ipsum dolor sit, amet consectetur adipisicing elit. Tenetur, molestiae perspiciatis nam quae libero, deserunt in aperiam unde officia sint saepe laboriosam ducimus aspernatur labore! Sapiente aspernatur corrupti quis ad.
                    </div>
                </template>
            </LocalVueUiTiremarks>
        </template>

        <template #VDUI-local>
            <LocalVueDataUi component="VueUiTiremarks" :dataset="isPropsToggled ? alternateDataset : dataset" :config="isPropsToggled ? alternateConfig : config" :key="`VDUI-lodal_${step}`">
            </LocalVueDataUi>
        </template>

        <template #build>
            <VueUiTiremarks :dataset="isPropsToggled ? alternateDataset : dataset" :config="isPropsToggled ? alternateConfig : config" :key="`build_${step}`">
            </VueUiTiremarks>
        </template>

        <template #VDUI-build>
            <VueDataUi component="VueUiTiremarks" :dataset="isPropsToggled ? alternateDataset : dataset" :config="isPropsToggled ? alternateConfig : config" :key="`VDUI-build_${step}`">
            </VueDataUi>
        </template>
        
        <template #knobs>
            <div
                style="display: flex; flex-direction: row; flex-wrap:wrap; align-items:center; width: 100%; color: #CCCCCC; gap:24px;">
                <div v-for="knob in model">
                    <label style="font-size: 10px">{{ knob.key }}</label>
                    <div
                        style="display:flex; flex-direction:row; flex-wrap: wrap; align-items:center; gap:6px; height: 40px">
                        <input v-if="!['none', 'select'].includes(knob.type)" :step="knob.step" :type="knob.type" :min="knob.min ?? 0"
                            :max="knob.max ?? 0" v-model="knob.def" @change="step += 1">
                        <select v-if="knob.type === 'select'" v-model="knob.def" @change="step += 1">
                            <option v-for="opt in knob.options">{{ opt }}</option>
                        </select>
                    </div>
                </div>
            </div>
        </template>

        <template #config>
            {{ config }}
        </template>
    </Box>
</template>