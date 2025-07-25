<script setup>
import { ref, computed, onMounted } from "vue";
import LocalVueUiDumbbell from '../src/components/vue-ui-dumbbell.vue';
import LocalVueDataUi from '../src/components/vue-data-ui.vue';
import Box from "./Box.vue";
import convertArrayToObject from "./convertModel";
import { useArena } from "../src/useArena";

const { local, build, vduiLocal, vduiBuild, toggleTable } = useArena()

const dataset =  ref([
    { name: 'Sweden', start: 5000, end: 7100 },
    { name: 'Korea, Rep.', start: 4900, end: 7050 },
    { name: 'Iceland', start: 6500, end: 8000 },
    { name: 'Finland', start: 6400, end: 7600 },
    { name: 'Norway', start: 5400, end: 6050 },
    { name: 'Ireland', start: 3000, end: 2000 }
])

onMounted(() => {
    setTimeout(() => {
        dataset.value.push({
            name: 'ALT',
            start: 3000,
            end: 3500
        })
    }, 3000)
})

const model = ref([
    { key: 'responsive', def: false, type: 'checkbox'},
    { key: 'userOptions.show', def: true, type: 'checkbox'},
    { key: 'userOptions.buttons.pdf', def: true, type: 'checkbox'},
    { key: 'userOptions.buttons.csv', def: true, type: 'checkbox'},
    { key: 'userOptions.buttons.img', def: true, type: 'checkbox'},
    { key: 'userOptions.buttons.table', def: true, type: 'checkbox'},
    { key: 'userOptions.buttons.fullscreen', def: true, type: 'checkbox'},
    { key: 'userOptions.position', def: 'right', type: 'select', options: ['left', 'right']},
    { key: 'userOptions.showOnChartHover', def: true, type: 'checkbox'},
    { key: 'userOptions.keepStateOnChartLeave', def: true, type: 'checkbox'},

    { key: 'userOptions.print.scale', def: 2, type: 'number', min: 1, max: 5},
    { key: 'userOptions.print.allowTaint', def: true, type: 'checkbox'},
    { key: 'userOptions.print.useCORS', def: true, type: 'checkbox'},
    { key: 'userOptions.print.backgroundColor', def: '#FFFFFF' },
    
    { key: 'useAnimation', def: true, type: 'checkbox'},
    { key: 'animationSpeed', def: 2, type: 'number', min: 1, max: 10},
    { key: 'style.fontFamily', def: 'inherit', type: 'text'},
    { key: 'style.chart.backgroundColor', def: '#FFFFFF20', type: 'color'},
    { key: 'style.chart.color', def: '#1A1A1A', type: 'color'},
    { key: 'style.chart.width', def: 600, type: 'number', min: 300, max: 1000},
    { key: 'style.chart.rowHeight', def: 40, type: 'number', min: 40, max: 100},
    { key: 'style.chart.padding.top', def: 12, type: 'number', min: 0, max: 100},
    { key: 'style.chart.padding.left', def: 100, type: 'number', min: 0, max: 100},
    { key: 'style.chart.padding.right', def: 24, type: 'number', min: 0, max: 100},
    { key: 'style.chart.padding.bottom', def: 12, type: 'number', min: 0, max: 100},
    { key: 'style.chart.plots.startColor', def: '#FF6400', type: 'color'},
    { key: 'style.chart.plots.endColor', def: '#5F8BEE', type: 'color'},
    { key: 'style.chart.plots.radius', def: 6, type: 'number', min: 2, max: 40},
    { key: 'style.chart.plots.stroke', def: '#FFFFFF', type: 'color'},
    { key: 'style.chart.plots.strokeWidth', def: 1, type: 'number', min: 0, max: 12},
    { key: 'style.chart.plots.link.strokeWidth', def: 2, type: 'number', min: 0, max: 12},
    { key: 'style.chart.plots.link.type', def: 'curved', type: 'select', options: ['curved', 'line']},
    { key: 'style.chart.plots.gradient.show', def: true, type: 'checkbox'},
    { key: 'style.chart.plots.gradient.intensity', def: 40, type: 'range', min: 0, max: 100},
    { key: 'style.chart.grid.strokeWidth', def: 1, type: 'number', min: 0, max: 12},
    { key: 'style.chart.grid.scaleSteps', def: 10, type: 'number', min: 2, max: 20},
    { key: 'style.chart.grid.horizontalGrid.show', def: true, type: 'checkbox'},
    { key: 'style.chart.grid.horizontalGrid.stroke', def: '#CCCCCC', type: 'color'},
    { key: 'style.chart.grid.horizontalGrid.strokeWidth', def: 0.5, type: 'number', min: 0, max: 12, step: 0.5},
    { key: 'style.chart.grid.horizontalGrid.strokeDasharray', def: 4, type: 'number', min: 0, max: 100},
    { key: 'style.chart.grid.verticalGrid.show', def: true, type: 'checkbox'},
    { key: 'style.chart.grid.verticalGrid.stroke', def: '#CCCCCC', type: 'color'},
    { key: 'style.chart.grid.verticalGrid.strokeWidth', def: 0.5, type: 'number', min: 0, max: 12, step: 0.5},
    { key: 'style.chart.grid.verticalGrid.strokeDasharray', def: 0, type: 'number', min: 0, max: 100},
    { key: 'style.chart.labels.prefix', def: 'P', type: 'text'},
    { key: 'style.chart.labels.suffix', def: 'S', type: 'text'},
    { key: 'style.chart.labels.yAxisLabels.show', def: true, type: 'checkbox'},
    { key: 'style.chart.labels.yAxisLabels.fontSize', def: 14, type: 'number', min: 8, max: 48},
    { key: 'style.chart.labels.yAxisLabels.color', def: '#1A1A1A', type: 'color'},
    { key: 'style.chart.labels.yAxisLabels.offsetX', def: 0, type: 'number', min: -100, max: 100},
    { key: 'style.chart.labels.yAxisLabels.bold', def: true, type: 'checkbox'},
    { key: 'style.chart.labels.yAxisLabels.showProgression', def: true, type: 'checkbox'},
    { key: 'style.chart.labels.yAxisLabels.rounding', def: 1, type: 'number', min: 0, max: 12},
    { key: 'style.chart.labels.xAxisLabels.show', def: true, type: 'checkbox'},
    { key: 'style.chart.labels.xAxisLabels.fontSize', def: 14, type: 'number', min: 8, max: 48},
    { key: 'style.chart.labels.xAxisLabels.color', def: '#1A1A1A', type: 'color'},
    { key: 'style.chart.labels.xAxisLabels.offsetY', def: 0, type: 'number', min: -100, max: 100},
    { key: 'style.chart.labels.xAxisLabels.bold', def: false, type: 'checkbox'},
    { key: 'style.chart.labels.xAxisLabels.rounding', def: 2, type: 'number', min: 0, max: 12},
    { key: 'style.chart.labels.startLabels.show', def: true, type: 'checkbox'},
    { key: 'style.chart.labels.startLabels.fontSize', def: 10, type: 'number', min: 8, max: 24},
    { key: 'style.chart.labels.startLabels.color', def: '#1A1A1A', type: 'color'},
    { key: 'style.chart.labels.startLabels.offsetY', def: 0, type: 'number', min: -100, max: 100},
    { key: 'style.chart.labels.startLabels.rounding', def: 2, type: 'number', min: 0, max: 12},
    { key: 'style.chart.labels.startLabels.useStartColor', def: true, type: 'checkbox'},
    { key: 'style.chart.labels.xAxisLabels.rounding', def: 2, type: 'number', min: 0, max: 12},
    { key: 'style.chart.labels.endLabels.show', def: true, type: 'checkbox'},
    { key: 'style.chart.labels.endLabels.fontSize', def: 10, type: 'number', min: 8, max: 24},
    { key: 'style.chart.labels.endLabels.color', def: '#1A1A1A', type: 'color'},
    { key: 'style.chart.labels.endLabels.offsetY', def: 0, type: 'number', min: -100, max: 100},
    { key: 'style.chart.labels.endLabels.rounding', def: 2, type: 'number', min: 0, max: 12},
    { key: 'style.chart.labels.endLabels.useEndColor', def: true, type: 'checkbox'},
    { key: 'style.chart.legend.show', def: true, type: 'checkbox'},
    { key: 'style.chart.legend.labelStart', def: 'start', type:'text'},
    { key: 'style.chart.legend.labelEnd', def: 'end', type: 'text'},
    { key: 'style.chart.legend.backgroundColor', def: '#FFFFFF', type: 'color'},
    { key: 'style.chart.legend.color', def: '#1A1A1A', type: 'color'},
    { key: 'style.chart.legend.fontSize', def: 14, type: 'number', min: 8, max: 48},
    { key: 'style.chart.legend.bold', def: false, type: 'checkbox'},
    { key: 'style.chart.title.text', def: 'Lorem ipsum dolor sit amet', type: 'text'},
    { key: 'style.chart.title.color', def: '#1A1A1A', type: 'color'},
    { key: 'style.chart.title.fontSize', def: 20, type: 'number', min: 8, max: 48},
    { key: 'style.chart.title.bold', def: true, type: 'checkbox'},
    { key: 'style.chart.title.subtitle.color', def: '#CCCCCC', type: 'color'},
    { key: 'style.chart.title.subtitle.text', def: 'Lorem ipsum dolor sit amet', type: 'text'},
    { key: 'style.chart.title.subtitle.fontSize', def: 16, type: 'number', min: 8, max: 48},
    { key: 'style.chart.title.subtitle.bold', def: false, type: 'checkbox'},
    { key: 'table.show', def: false, type: 'checkbox'},
    { key: 'table.responsiveBreakpoint', def: 300, type: 'number', min: 300, max: 800},
    { key: 'table.columnNames.series', def: 'Series', type: 'text'},
    { key: 'table.columnNames.start', def: 'Start value', type: 'text'},
    { key: 'table.columnNames.end', def: 'End value', type: 'text'},
    { key: 'table.columnNames.progression', def: 'Progression', type: 'text'},
    { key: 'table.th.backgroundColor', def: '#FFFFFF', type: 'color'},
    { key: 'table.th.color', def:'#1A1A1A', type: 'color'},
    { key: 'table.th.outline', def: 'none', type: 'text'},
    { key: 'table.td.backgroundColor', def: '#FFFFFF', type: 'color'},
    { key: 'table.td.color', def:'#1A1A1A', type: 'color'},
    { key: 'table.td.outline', def: 'none', type: 'text'},
    { key: 'table.td.roundingValue', def: 2, type: 'number', min: 0, max:12},
    { key: 'table.td.roundingPercentage', def: 2, type: 'number', min: 0, max:12},
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
                labels: {
                    ...c.style.chart.labels,
                    formatter: ({value, config}) => {
                        // console.log(config)
                        return `f | ${value}`
                    }
                }
            }
        },
        theme: currentTheme.value
    }
})

const step = ref(0);

onMounted(async () => {
    if (local.value) {
        const img = await local.value.getImage()
        console.log(img)
    }
})

</script>

<template>
    <button @click="toggleTable">TOGGLE TABLE</button>

    <div style="margin: 12px 0; color: white">
        Theme:
        <select v-model="currentTheme" @change="step += 1">
            <option v-for="opt in themeOptions">{{ opt }}</option>
        </select>
    </div>

    <div style="width: 600px; height: 600px; resize: both; overflow: auto; background: white">
        <LocalVueUiDumbbell :key="`responsive_${step}`" :dataset="dataset" :config="{
            ...config,
            responsive: true
        }">
        <template #chart-background>
            <div style="width: 100%; height: 100%; background: radial-gradient(at top left, red, white)"/>
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
    </LocalVueUiDumbbell>
    </div>

    <Box comp="VueUiDumbbell" :dataset="dataset">
        <template #title>VueUiDumbbell</template>
        <template #local>
            <LocalVueUiDumbbell :dataset="dataset" :config="config" :key="`local_${step}`" ref="local">
                <template #optionPdf>
                    PRINT PDF
                </template>
                <template #svg="{ svg }">
                    <circle :cx="30" :cy="30" :r="30" fill="#42d392" />
                    <text :x="30" :y="30" text-anchor="middle">#SVG</text>
                </template>
                <template #legend="{ legend }">
                    #LEGEND
                    <div style="font-size: 8px">
                        {{ legend }}
                    </div>
                </template>
            </LocalVueUiDumbbell>
        </template>

        <template #VDUI-local>
            <LocalVueDataUi component="VueUiDumbbell" :dataset="dataset" :config="config" :key="`vdui_local_${step}`" ref="vduiLocal">
                <template #svg="{ svg }">
                    <circle :cx="30" :cy="30" :r="30" fill="#42d392" />
                    <text :x="30" :y="30" text-anchor="middle">#SVG</text>
                </template>
                <template #legend="{ legend }">
                    #LEGEND
                    <div style="font-size: 8px">
                        {{ legend }}
                    </div>
                </template>
            </LocalVueDataUi>
        </template>

        <template #build>
            <VueUiDumbbell :dataset="dataset" :config="config" :key="`build_${step}`" ref="build">
                <template #svg="{ svg }">
                    <circle :cx="30" :cy="30" :r="30" fill="#42d392" />
                    <text :x="30" :y="30" text-anchor="middle">#SVG</text>
                </template>
                <template #legend="{ legend }">
                    #LEGEND
                    <div style="font-size: 8px">
                        {{ legend }}
                    </div>
                </template>
            </VueUiDumbbell>
        </template>

        <template #VDUI-build>
            <VueDataUi component="VueUiDumbbell" :dataset="dataset" :config="config" :key="`vdui_build_${step}`" ref="vduiBuild">
                <template #svg="{ svg }">
                    <circle :cx="30" :cy="30" :r="30" fill="#42d392" />
                    <text :x="30" :y="30" text-anchor="middle">#SVG</text>
                </template>
                <template #legend="{ legend }">
                    #LEGEND
                    <div style="font-size: 8px">
                        {{ legend }}
                    </div>
                </template>
            </VueDataUi>
        </template>

        <template #knobs>
            <div
                style="display: flex; flex-direction: row; flex-wrap:wrap; align-items:center; width: 100%; color: #CCCCCC; gap:24px;">
                <div v-for="knob in model">
                    <label style="font-size: 10px">{{ knob.key }}</label>
                    <div
                        style="display:flex; flex-direction:row; flex-wrap: wrap; align-items:center; gap:6px; height: 40px">
                        <input v-if="!['none', 'select'].includes(knob.type)" :step="knob.step" :type="knob.type"
                            :min="knob.min ?? 0" :max="knob.max ?? 0" v-model="knob.def" @change="step += 1">
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