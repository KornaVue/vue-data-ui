<script setup>
import { ref, computed, onMounted } from "vue";
import LocalVueUiScatter from '../src/components/vue-ui-scatter.vue';
import LocalVueDataUi from '../src/components/vue-data-ui.vue';
import Box from "./Box.vue";
import convertArrayToObject from "./convertModel";
import { useArena } from "../src/useArena";

const { local, build, vduiLocal, vduiBuild, toggleTable } = useArena()

const scat1 = computed(() => {
    const arr = [];
    for (let i = -100; i < 100; i += 1) {
        arr.push({
            x: Math.random() * (Math.random() > 0.3 ? i / 3 : -i / 5),
            y: Math.random() * i / 20,
            name: `plot_${i}_cluster_1`
        });
    }
    return arr;
});

const scat2 = computed(() => {
    const arr = [];
    for (let i = -100; i < 100; i += 1) {
        arr.push({
            x: Math.random() * (Math.random() > 0.1 ? i / 10 : -i / 10),
            y: Math.random() * i / 10,
            name: `plot_${i}_cluster_2`,
        });
    }
    return arr;
});

const dataset = computed(() => {

    return [
        {
            name: "Cluster 1",
            values: scat1.value,
            shape: "star",
        },
        {
            name: "Cluster 2",
            values: scat2.value,
            shape: "triangle",
            color: 'orange'
        }
    ]
});

const alternateDataset = ref([
    {
            name: 'Alt',
            values: [
                {
                    x: 0,
                    y: 0,
                    name: 'alt1'
                },
                {
                    x: 1,
                    y: 1,
                    name: 'alt1'
                }
            ],
            shape: 'pentagon'
    }
])

const alternateConfig = ref({
    table: {
        th: {
            backgroundColor: '#00FF00'
        }
    },
    style: {
        backgroundColor: '#CCCCCC',
        title: {
            text: 'Alternate'
        }
    }
})

const isPropsToggled = ref(false);
function toggleProps() {
    isPropsToggled.value = !isPropsToggled.value;
}

const trigger = ref(0);

function alterDataset() {
    alternateDataset.value[0].values.push({
        x: trigger.value % 5 === 0 ? Math.random()*10 : Math.random() * -10,
        y: trigger.value % 5 === 0 ? Math.random()*10 : Math.random() * -10
    })
    trigger.value += 1;
}

const model = ref([
    { key: 'responsive', def: false, type: 'checkbox'},
    { key: 'userOptions.show', def: true, type: 'checkbox'},
    { key: 'userOptions.buttons.pdf', def: true, type: 'checkbox'},
    { key: 'userOptions.buttons.img', def: true, type: 'checkbox'},
    { key: 'userOptions.buttons.csv', def: true, type: 'checkbox'},
    { key: 'userOptions.buttons.table', def: true, type: 'checkbox'},
    { key: 'userOptions.buttons.fullscreen', def: true, type: 'checkbox'},
    { key: 'userOptions.position', def: 'right', type: 'select', options: ['left', 'right']},
    { key: 'userOptions.showOnChartHover', def: true, type: 'checkbox'},
    { key: 'userOptions.keepStateOnChartLeave', def: true, type: 'checkbox'},

    { key: 'userOptions.print.scale', def: 2, type: 'number', min: 1, max: 5},
    { key: 'userOptions.print.allowTaint', def: true, type: 'checkbox'},
    { key: 'userOptions.print.useCORS', def: true, type: 'checkbox'},
    { key: 'userOptions.print.backgroundColor', def: '#FFFFFF' },

    { key: 'useCssAnimation', def: true, type: 'checkbox'},
    { key: 'style.fontFamily', def: "inherit", type: 'text'},
    { key: 'style.backgroundColor', def: '#FFFFFF20', type: 'color'},
    { key: 'style.color', def: '#1A1A1A', type: 'color'},
    { key: 'style.layout.height', def: 316, type: 'number', min: 100, max: 1000},
    { key: 'style.layout.width', def: 512, type: 'number', min: 100, max: 1000},
    { key: 'style.layout.padding.top', def: 36, type: 'number', min: 0, max: 100},
    { key: 'style.layout.padding.right', def: 48, type: 'number', min: 0, max: 100},
    { key: 'style.layout.padding.bottom', def: 36, type: 'number', min: 0, max: 100},
    { key: 'style.layout.padding.left', def: 48, type: 'number', min: 0, max: 100},
    { key: 'style.layout.axis.show', def: true, type: 'checkbox'},
    { key: 'style.layout.axis.stroke', def: '#e1e5e8', type: 'color'},
    { key: 'style.layout.axis.strokeWidth', def: 1, type: 'number', min: 0, max: 12},
    { key: 'style.layout.marginalBars.show', def: true, type: 'checkbox'},
    { key: 'style.layout.marginalBars.size', def: 40, type: 'number', min:12, max: 100},
    { key: 'style.layout.marginalBars.tranches', def: 20, type: 'number', min: 5, max: 100},
    { key: 'style.layout.marginalBars.opacity', def: 0.6, type: 'range', min: 0, max: 1, step: 0.01},
    { key: 'style.layout.marginalBars.fill', def: '#1A1A1A', type: 'color'},
    { key: 'style.layout.marginalBars.strokeWidth', def: 1, type: 'number', min: 0, max: 12},
    { key: 'style.layout.marginalBars.offset', def: 20, type: 'number', min: 0, max: 100},
    { key: 'style.layout.marginalBars.borderRadius', def: 2, type: 'number', min: 0, max: 24},
    { key: 'style.layout.marginalBars.useGradient', def: true, type: 'checkbox'},
    { key: 'style.layout.marginalBars.showLines', def: true, type: 'checkbox'},
    { key: 'style.layout.marginalBars.linesStrokeWidth', def: 1, type: 'number', min: 0.5, max: 12, step: 0.5},
    { key: 'style.layout.plots.radius', def: 2, type: 'number', min: 0, max: 24},
    { key: 'style.layout.plots.stroke', def: '#FFFFFF', type: 'color'},
    { key: 'style.layout.plots.strokeWidth', def: 0.3, type: 'range', min: 0.1, max: 12, step: 0.1},
    { key: 'style.layout.plots.opacity', def: 0.6, type: 'number', min: 0, max: 1, step: 0.01},
    { key: 'style.layout.plots.significance.show', def: true, type: 'checkbox'},
    { key: 'style.layout.plots.significance.useDistanceOpacity', def: true, type: 'checkbox' },
    { key: 'style.layout.plots.significance.deviationThreshold', def: 10, type: 'number', min: 0, max: 100},
    { key: 'style.layout.plots.significance.opacity', def: 0.3, type: 'number', min: 0, max: 1, step: 0.01},
    { key: 'style.layout.plots.deviation.translation', def: 'deviation', type: 'text'},
    { key: 'style.layout.plots.deviation.roundingValue', def: 1, type: 'number', min: 0, max: 12},
    { key: 'style.layout.plots.giftWrap.show', def: false, type: 'checkbox'},
    { key: 'style.layout.plots.giftWrap.strokeWidth', def: 1, type: 'number', min: 0, max: 12},
    { key: 'style.layout.plots.giftWrap.strokeDasharray', def: 0, type: 'number', min: 0, max: 100},
    { key: 'style.layout.plots.giftWrap.fillOpacity', def: 0.2, type: 'number', min: 0, max: 1, step: 0.01},
    { key: 'style.layout.plots.selectors.show', def: true, type: 'checkbox'},
    { key: 'style.layout.plots.selectors.stroke', def: '#1A1A1A', type: 'color'},
    { key: 'style.layout.plots.selectors.strokeWidth', def: 0.7, type: 'number', min: 0, max: 12, step: 0.1},
    { key: 'style.layout.plots.selectors.strokeDasharray', def: 0, type: 'number', min: 0, max: 100},
    { key: 'style.layout.plots.selectors.labels.fontSize', def: 12, type: 'number', min: 8, max: 24},
    { key: 'style.layout.plots.selectors.labels.color', def: '#1A1A1A', type: 'color'},
    { key: 'style.layout.plots.selectors.labels.rounding', def: 2, type: 'number', min: 0, max: 12},
    { key: 'style.layout.plots.selectors.labels.bold', def: false, type: 'checkbox'},
    { key: 'style.layout.plots.selectors.labels.showName', def: true, type: "checkbox"},
    { key: 'style.layout.plots.selectors.labels.prefix', def: 'P', type: 'text'},
    { key: 'style.layout.plots.selectors.labels.suffix', def: 'S', type: 'text'},
    { key: 'style.layout.plots.selectors.markers.radius', def: 1.5, type: 'number', min: 0, max: 12, step: 0.5},
    { key: 'style.layout.plots.selectors.markers.stroke', def: '#FFFFFF', type: 'color'},
    { key: 'style.layout.plots.selectors.markers.strokeWidth', def: 0.5, type: 'number', min: 0, max: 12, step: 0.5},
    { key: 'style.layout.plots.selectors.markers.fill', def: '#1A1A1A', type: 'color'},
    { key: 'style.layout.correlation.show', def: true, type: 'checkbox'},
    { key: 'style.layout.correlation.strokeDasharray', def: 2, type: 'number', min: 0, max: 100},
    { key: 'style.layout.correlation.strokeWidth', def: 1, type: 'number', min: 0, max: 12},
    { key: 'style.layout.correlation.label.show', def: true, type: 'checkbox'},
    { key: 'style.layout.correlation.label.fontSize', def: 12, type: 'number', min: 8, max: 48},
    { key: 'style.layout.correlation.label.color', def: '#1A1A1A', type: 'color'},
    { key: 'style.layout.correlation.label.bold', def: true, type: 'checkbox'},
    { key: 'style.layout.correlation.label.roundingValue', def: 2, type: 'number', min: 0, max: 12},
    { key: 'style.layout.correlation.label.useSerieColor', def: true, type: 'checkbox'},
    { key: 'style.layout.dataLabels.xAxis.name', def: 'Lorem Ipsum X', type: 'text'},
    { key: 'style.layout.dataLabels.xAxis.show', def: true, type: 'checkbox'},
    { key: 'style.layout.dataLabels.xAxis.fontSize', def: 8, type: 'number', min: 8, max: 24},
    { key: 'style.layout.dataLabels.xAxis.color', def: '#1A1A1A', type: 'color'},
    { key: 'style.layout.dataLabels.xAxis.bold', def: false, type: 'checkbox'},
    { key: 'style.layout.dataLabels.xAxis.offsetX', def: 0, type: 'number', min: -100, max: 100},
    { key: 'style.layout.dataLabels.xAxis.offsetY', def: 0, type: 'number', min: -100, max: 100},
    { key: 'style.layout.dataLabels.xAxis.roundingValue', def: 2, type: 'number', min: 0, max: 12},
    { key: 'style.layout.dataLabels.yAxis.name', def: 'Lorem Ipsum Y', type: 'text'},
    { key: 'style.layout.dataLabels.yAxis.show', def: true, type: 'checkbox'},
    { key: 'style.layout.dataLabels.yAxis.fontSize', def: 8, type: 'number', min: 8, max: 24},
    { key: 'style.layout.dataLabels.yAxis.color', def: '#1A1A1A', type: 'color'},
    { key: 'style.layout.dataLabels.yAxis.bold', def: false, type: 'checkbox'},
    { key: 'style.layout.dataLabels.yAxis.offsetX', def: 0, type: 'number', min: -100, max: 100},
    { key: 'style.layout.dataLabels.yAxis.offsetY', def: 0, type: 'number', min: -100, max: 100},
    { key: 'style.layout.dataLabels.yAxis.roundingValue', def: 2, type: 'number', min: 0, max: 12},
    { key: 'style.title.text', def: 'At vero eos et accusamus et iusto odio dignissimos ducimus qui blanditiis', type: 'text'},
    { key: 'style.title.color', def: '#1A1A1A', type: 'color'},
    { key: 'style.title.fontSize', def: 20, type: 'number', min: 8, max: 48},
    { key: 'style.title.bold', def: true, type: 'checkbox'},
    { key: 'style.title.subtitle.text', def: 'At vero eos et accusamus et iusto odio dignissimos ducimus qui blanditiis', type: 'text'},
    { key: 'style.title.subtitle.color', def: '#CCCCCC', type: 'color'},
    { key: 'style.title.subtitle.fontSize', def: 16, type: 'range', min: 8, max: 48},
    { key: 'style.title.subtitle.bold', def: false, type: 'checkbox'},
    { key: 'style.legend.show', def: true, type: 'checkbox' },
    { key: 'style.legend.backgroundColor', def: '#FFFFFF20', type: 'color'},
    { key: 'style.legend.color', def: '#1A1A1A', type: 'color'},
    { key: 'style.legend.fontSize', def: 14, type: 'number', min: 6, max: 42 },
    { key: 'style.legend.bold', def: false, type: 'checkbox'},
    { key: 'style.legend.roundingValue', def: 0, type: 'number', min: 0, max: 6},

    { key: 'style.tooltip.show', def: true, type: 'checkbox' },
    { key: 'style.tooltip.backgroundColor', def: '#FFFFFF', type: 'color' },
    { key: 'style.tooltip.color', def: '#1A1A1A', type: 'color' },
    { key: 'style.tooltip.fontSize', def: 14, type: 'number', min: 6, max: 24 },
    { key: 'style.tooltip.showValue', def: true, type: 'checkbox'},
    { key: 'style.tooltip.roundingValue', def: 2, type: 'number', min: 0, max: 6},
    { key: 'style.tooltip.showShape', def: true, type: 'checkbox' },
    { key: 'style.tooltip.prefix', def: 'P', type: 'text'},
    { key: 'style.tooltip.suffix', def: 'S', type: 'text'},
    { key: 'style.tooltip.backgroundOpacity', def: 100, type: 'range', min: 0, max: 100},
    { key: 'style.tooltip.position', def: 'center', type: 'select', options: ['left', 'center', 'right']},
    { key: 'style.tooltip.offsetY', def: 24, type: 'number', min: 0, max: 48 },

    { key: 'table.show', def: false, type: 'checkbox', label: 'show', category: 'table' },
    { key: 'table.responsiveBreakpoint', def: 400, type: 'number', min: 300, max: 800 },
    { key: 'table.th.backgroundColor', def: '#FFFFFF', type: 'color' },
    { key: 'table.th.color', def: '#1A1A1A', type: 'color' },
    { key: 'table.th.outline', def: 'none', type: 'text' },
    { key: 'table.td.backgroundColor', def: '#FFFFFF', type: 'color' },
    { key: 'table.td.color', def: '#1A1A1A', type: 'color'},
    { key: 'table.td.outline', def: 'none', type: 'text' },
    { key: 'table.td.roundingValue', def: 2, type: 'number', min: 0, max: 6 },
    { key: 'table.td.roundingAverage', def: 1, type: 'number', min: 0, max: 6 },
    { key: 'table.translations.correlationCoefficient', def: 'Correlation coef.', type: 'text'},
    { key: 'table.translations.nbrPlots', def: 'Nbr plots', type: 'text'},
    { key: 'table.translations.average', def: 'Average', type: 'text'},
    { key: 'table.translations.series', def: 'Series', type: 'text'},
])

const testCustomTooltip = ref(false);

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
    if(testCustomTooltip.value) {
        return {
            ...c,
            style: {
                ...c.style,
                tooltip: {
                    ...c.style.tooltip,
                    customFormat: ({ datapoint }) => {
                        let html = '';
                        // console.log(datapoint);
                        return "test"
                    }
                }
            }

        }
    } else {
        return {
            ...c,
            style: {
                ...c.style,
                layout: {
                    ...c.style.layout,
                    plots: {
                        ...c.style.layout.plots,
                        selectors: {
                            ...c.style.layout.plots.selectors,
                            labels: {
                                ...c.style.layout.plots.selectors.labels,
                                x: {
                                    formatter: ({value, config}) => {
                                        // console.log(config)
                                        return `X | ${value}`
                                    }
                                },
                                y: {
                                    formatter: ({value, config}) => {
                                        // console.log(config)
                                        return `Y | ${value}`
                                    }
                                }
                            }
                        }
                    }
                }
            },
            theme: currentTheme.value,
            customPalette: ['#6376DD', "#DD3322", "#66DDAA"],
        }
    }
});

const step = ref(0)

function selectLegend(legend) {
    console.log({legend})
}

onMounted(async () => {
    if (local.value) {
        const img = await local.value.getImage()
        console.log(img)
    }
})

</script>

<template>
    <button @click="toggleTable">TOGGLE TABLE</button>
    <div style="margin: 12px 0">
        <input type="checkbox" v-model="testCustomTooltip" id="custom-tooltip" />
        <label for="custom-tooltip" style="color:#CCCCCC">Test custom tooltip</label>
    </div>

    <button @click="toggleProps">TOGGLE PROPS: {{ isPropsToggled }}</button>
    <button @click="alterDataset">ALTER DATASET</button>

    <div style="width: 600px; height: 600px; resize: both; overflow: auto; background: white">
        <LocalVueUiScatter :key="`responsive_${step}`" :dataset="dataset" :config="{
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
    </LocalVueUiScatter>
    </div>

    <Box comp="VueUiScatter" :dataset="dataset">
        <template #title>VueUiScatter</template>
        
        <template #local>
            <LocalVueUiScatter :dataset="isPropsToggled ? alternateDataset : dataset" :config="isPropsToggled ? alternateConfig : config" :key="`local_${step}`" ref="local">
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
                <template #tooltip-before="{ datapoint, seriesIndex, series, config, bars, lines, plots }">
                    #BEFORE {{ series.name }}
                </template>
                <template #tooltip-after="{ datapoint, seriesIndex, series, config, bars, lines, plots }">
                    #AFTER {{ series.name }}
                </template>
                <template #watermark="{ isPrinting }">
                    <div v-if="isPrinting" style="font-size: 100px; opacity: 0.1; transform: rotate(-10deg)">
                        WATERMARK
                    </div>
                </template>
            </LocalVueUiScatter>
        </template>

        <template #VDUI-local>
            <LocalVueDataUi component="VueUiScatter" :dataset="isPropsToggled ? alternateDataset : dataset" :config="isPropsToggled ? alternateConfig : config" :key="`VDUI-lodal_${step}`" ref="vduiLocal">
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
                <template #tooltip-before="{ datapoint, seriesIndex, series, config, bars, lines, plots }">
                    #BEFORE {{ series.name }}
                </template>
                <template #tooltip-after="{ datapoint, seriesIndex, series, config, bars, lines, plots }">
                    #AFTER {{ series.name }}
                </template>
            </LocalVueDataUi>
        </template>

        <template #build>
            <VueUiScatter :dataset="isPropsToggled ? alternateDataset : dataset" :config="isPropsToggled ? alternateConfig : config" :key="`build_${step}`" ref="build">
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
                <template #tooltip-before="{ datapoint, seriesIndex, series, config, bars, lines, plots }">
                    #BEFORE {{ series.name }}
                </template>
                <template #tooltip-after="{ datapoint, seriesIndex, series, config, bars, lines, plots }">
                    #AFTER {{ series.name }}
                </template>
            </VueUiScatter>
        </template>

        <template #VDUI-build>
            <VueDataUi component="VueUiScatter" :dataset="isPropsToggled ? alternateDataset : dataset" :config="isPropsToggled ? alternateConfig : config" :key="`VDUI-build_${step}`" ref="vduiBuild">
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
                <template #tooltip-before="{ datapoint, seriesIndex, series, config, bars, lines, plots }">
                    #BEFORE {{ series.name }}
                </template>
                <template #tooltip-after="{ datapoint, seriesIndex, series, config, bars, lines, plots }">
                    #AFTER {{ series.name }}
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