<script setup>
import { ref, computed, onMounted, nextTick, watch, useSlots, defineAsyncComponent, onBeforeUnmount, shallowRef } from "vue";
import { useNestedProp } from "../useNestedProp";
import {
    applyDataLabel,
    calcMedian, 
    convertColorToHex, 
    convertCustomPalette, 
    createCsvContent, 
    createUid, 
    dataLabel, 
    downloadCsv,
    error,
    getMissingDatasetAttributes,
    objectIsEmpty,
    palette,
    themePalettes,
} from "../lib";
import { usePrinter } from "../usePrinter";
import { useConfig } from "../useConfig";
import { useUserOptionState } from "../useUserOptionState";
import vClickOutside from "../directives/vClickOutside";
import themes from "../themes.json";

const SparkLine = defineAsyncComponent(() => import('./vue-ui-sparkline.vue'));
const BaseIcon = defineAsyncComponent(() => import('../atoms/BaseIcon.vue'));
const UserOptions = defineAsyncComponent(() => import('../atoms/UserOptions.vue'));

const { vue_ui_table_sparkline: DEFAULT_CONFIG } = useConfig();

const props = defineProps({
    config: {
        type: Object,
        default() {
            return {};
        },
    },
    dataset: {
        type: Array,
        default() {
            return [];
        },
    },
});

const uid = ref(createUid());
const step = ref(0);
const sparkStep = ref(0);
const TD = ref(null);
const slots = useSlots();

onMounted(() => {
    if (slots['chart-background']) {
        console.warn('VueUiTableSparkline does not support the #chart-background slot.')
    }
});

const FINAL_CONFIG = computed({
    get: () => {
        return prepareConfig();
    },
    set: (newCfg) => {
        return newCfg
    }
});

const { userOptionsVisible, setUserOptionsVisibility, keepUserOptionState } = useUserOptionState({ config: FINAL_CONFIG.value });

function prepareConfig() {
    const mergedConfig = useNestedProp({
        userConfig: props.config,
        defaultConfig: DEFAULT_CONFIG
    });
    if (mergedConfig.theme) {
        return {
            ...useNestedProp({
                userConfig: themes.vue_ui_table_sparkline[mergedConfig.theme] || props.config,
                defaultConfig: mergedConfig
            }),
            customPalette: themePalettes[mergedConfig.theme] || palette
        }
    } else {
        return mergedConfig;
    }
}

watch(() => props.config, (_newCfg) => {
    FINAL_CONFIG.value = prepareConfig();
    userOptionsVisible.value = !FINAL_CONFIG.value.userOptions.showOnChartHover;
    prepareChart();
    sparkStep.value += 1;
}, { deep: true });

watch(() => props.dataset, (_) => {
    mutableDataset.value = datasetWithOrders.value;
    sparkStep.value += 1;
}, { deep: true })

const { isPrinting, isImaging, generatePdf, generateImage } = usePrinter({
    elementId: `table_${uid.value}`,
    fileName: FINAL_CONFIG.value.title.text || 'vue-ui-table-sparkline',
    options: FINAL_CONFIG.value.userOptions.print
});

const customPalette = computed(() => {
    return convertCustomPalette(FINAL_CONFIG.value.customPalette);
})

const tableContainer = ref(null);
const isResponsive = ref(false);
const breakpoint = computed(() => {
    return FINAL_CONFIG.value.responsiveBreakpoint;
});

onMounted(() => {
    prepareChart();
});

const usableColNames = ref(FINAL_CONFIG.value.colNames)

const tableObserver = shallowRef(null);

function prepareChart() {
    if(objectIsEmpty(props.dataset)) {
        error({
            componentName: 'VueUiTableSparkline',
            type: 'dataset'
        })
    }

    if (tableObserver.value) {
        tableObserver.value.disconnect();
    }

    tableObserver.value = new ResizeObserver((entries) => {
        entries.forEach((entry) => {
            isResponsive.value = entry.contentRect.width < breakpoint.value;
        });
    });
    if (tableContainer.value) {
        tableObserver.value.observe(tableContainer.value);
    }
    usableColNames.value = [];

    for(let i =0; i < maxSeriesLength.value; i += 1) {
        usableColNames.value.push(FINAL_CONFIG.value.colNames[i] || `col ${i}`)
    }
}

onBeforeUnmount(() => {
    if (tableObserver.value) {
        tableObserver.value.disconnect();
    }
});

const computedDataset = computed(() => {
    props.dataset.forEach((ds, i) => {
        getMissingDatasetAttributes({
            datasetObject: ds,
            requiredAttributes: ['name', 'values']
        }).forEach(attr => {
            error({
                componentName: 'VueUiTableSparkline',
                type: 'datasetSerieAttribute',
                property: attr,
                index: i
            });
        });
    });

    return props.dataset.map((ds, i) => {
        const cleanValues = (ds.values || []).map((v) => (isNaN(v) ? 0 : v ?? 0));
        const sum = cleanValues.reduce((a, b) => a + b, 0);
        const average = sum / cleanValues.length;
        const median = calcMedian(cleanValues);
        return {
            ...ds,
            values: ds.values || [],
            color: convertColorToHex(ds.color) || customPalette.value[i] || palette[i] || palette[i % palette.length],
            sum,
            average,
            median,
            sparklineDataset: cleanValues.map((v, i) => {
                return {
                    period: FINAL_CONFIG.value.colNames[i] || `col ${i}`,
                    value: v || 0,
                };
            }),
        };
    });
});

function addOrdersAttribute(dataset) {
    const combinedValues = (dataset[0].values || []).map((_, index) =>
        dataset.map((series) => (series.values[index] || []))
    );

    const orders = combinedValues.map((values) =>
        values
            .map((value, index) => [value, index])
            .sort((a, b) => b[0] - a[0])
            .map((item) => item[1])
    );

    const result = dataset.map((series, index) => ({
        ...series,
        values: series.values || [],
        orders: orders[index],
    }));

    return result;
}

const datasetWithOrders = computed(() => {
    return addOrdersAttribute(computedDataset.value);
});

const mutableDataset = ref(datasetWithOrders.value)

const maxSeries = computed(() =>{
    return Math.max(...mutableDataset.value.map(ds => (ds.values || []).length))
})

const sortIndex = ref(undefined)
const isSorting = ref(false);
const currentSortingIndex = ref(undefined);
const currentSortOrder = ref(1);

function restoreOrder() {
    isSorting.value = false;
    currentSortingIndex.value = undefined;
    currentAdditionalSort.value = undefined;
    currentSortOrder.value = 1;
    sortStates.value.forEach(c => c.state = 1)
    sortOrders.value = {
        name: 1,
        sum: 1,
        average: 1,
        median: 1
    }
    mutableDataset.value = datasetWithOrders.value;
}

function orderDatasetByIndex(th, index, order) {

    if ((['name', 'sum', 'average', 'median'].includes(th.type))) {
        orderDatasetByAttribute(th.type, index, order);
        return;
    }

    if (!hasSorting(index)) return;

    selectedDataIndex.value = index;
    currentAdditionalSort.value = undefined;

    if ((![null, undefined].includes(currentSortingIndex.value) && index !== currentSortingIndex.value)) {
        currentSortingIndex.value = undefined;
        restoreOrder();
    }

    if (sortStates.value[index].state === order && currentSortingIndex.value === index) {
        currentSortingIndex.value = undefined;
        restoreOrder();
        return;
    }

    isSorting.value = true;
    currentSortingIndex.value = index;

    const combinedValues = datasetWithOrders.value.map(series => (series.values[index] || []));

    const sortOrder = order;
    sortStates.value[index].state = sortOrder;

    currentSortOrder.value = sortOrder;
    if(index === sortIndex.value) {
        sortIndex.value = undefined
    }else {
        sortIndex.value = index;
    } 

    const sortedIndices = combinedValues
        .map((value, i) => [i, value])
        .sort((a, b) => sortOrder * (b[1] - a[1]))
        .map(item => item[0]);

    const sortedDataset = sortedIndices.map(i => datasetWithOrders.value[i]);

    mutableDataset.value = sortedDataset;
    sparkStep.value += 1
}

const maxSeriesLength = computed(() => {
    return Math.max(...props.dataset.map(ds => (ds.values || []).length))
})

const colNames = computed(() => {
    let cn = usableColNames.value.map(c => {
        return {
            type: 'reg',
            value: c
        }
    });

    if(!cn.length) {
        for(let i = 0; i < maxSeriesLength.value; i += 1) {
            cn.push({type: 'reg', value: `col ${i+1}`})
        }
    }

    if (FINAL_CONFIG.value.showTotal) {
        cn = [...cn, { type: 'sum', value: FINAL_CONFIG.value.translations.total}];
    }

    let res;
    if (FINAL_CONFIG.value.showAverage && FINAL_CONFIG.value.showMedian) {
        res = [
            ...cn,
            { type: 'average', value: FINAL_CONFIG.value.translations.average},
            { type: 'median', value: FINAL_CONFIG.value.translations.median}
        ];
    } else if (FINAL_CONFIG.value.showAverage && !FINAL_CONFIG.value.showMedian) {
        res = [...cn, { type: 'average', value: FINAL_CONFIG.value.translations.average}];
    } else if (!FINAL_CONFIG.value.showAverage && FINAL_CONFIG.value.showMedian) {
        res = [...cn, { type: 'median', value: FINAL_CONFIG.value.translations.median}];
    } else {
        res = cn;
    }
    if (FINAL_CONFIG.value.showSparklines) {
        return [...res, { type: 'chart', value: FINAL_CONFIG.value.translations.chart}];
    } else {
        return res;
    }
});

const sortOrders = ref({
    name: 1,
    sum: 1,
    average: 1,
    median: 1
})

const currentAdditionalSort = ref(undefined);

function orderDatasetByAttribute(attribute, index, order) {
    if (!mutableDataset.value || mutableDataset.value.length === 0) return;
    if (!hasAdditionalSorting(attribute)) return;

    if (currentAdditionalSort.value !== attribute) {
        currentAdditionalSort.value = undefined;
    }

    if (![null, undefined].includes(currentSortingIndex.value) && index !== currentSortingIndex.value) {
        restoreOrder();
    }

    currentSortingIndex.value = undefined;


    if (sortOrders.value[attribute] === order && currentAdditionalSort.value) {
        currentAdditionalSort.value = undefined;
        restoreOrder();
        return;
    }

    currentAdditionalSort.value = attribute;
    isSorting.value = true;

    sortOrders.value[attribute] = order;

    if (![null, undefined].includes(index)) {
        sortStates.value[index].state = sortOrders.value[attribute];
    }

    const sortOrder = sortOrders.value[attribute];

    const sortedDataset = [...mutableDataset.value].sort((a, b) => {
        const aValue = a[attribute] ?? (typeof a[attribute] === 'number' ? 0 : '');
        const bValue = b[attribute] ?? (typeof b[attribute] === 'number' ? 0 : '');

        if (typeof aValue === 'string' && typeof bValue === 'string') {
            return sortOrder === -1 ? aValue.localeCompare(bValue) : bValue.localeCompare(aValue);
        } else if (typeof aValue === 'number' && typeof bValue === 'number') {
            return sortOrder === -1 ? aValue - bValue : bValue - aValue;
        }
        return 0;
    });
    mutableDataset.value = sortedDataset;
}

const selectedDataIndex = ref(undefined);
const selectedSerieIndex = ref(undefined);

function hoverSparkline({ dataIndex, serieIndex }) {
    selectedDataIndex.value = dataIndex;
    selectedSerieIndex.value = serieIndex;
    if (TD.value[dataIndex] && !isResponsive.value) {
        TD.value[dataIndex].scrollIntoView({ behavior: 'smooth', block: 'nearest', inline: 'center' });
    }
}

const isFullscreen = ref(false)
function toggleFullscreen(state) {
    isFullscreen.value = state;
    step.value += 1;
}

function generateCsv(callback=null) {
    nextTick(() => {
        const thead = [FINAL_CONFIG.value.translations.serie].concat(colNames.value)
        const tbody = computedDataset.value.map((ds, i) => {
            return [
                [ds.name],
                ds.values,
                [ds.sum],
                [ds.average],
                [ds.median]
            ]
        });
        const t = [thead].concat(tbody)
        const csvContent = createCsvContent(t)

        if (!callback) {
            downloadCsv({
                csvContent,
                title: FINAL_CONFIG.value.title.text || "vue-ui-table-sparkline"
            })
        } else {
            callback(csvContent);
        }
    })
}

function hasSorting(index) {
    return FINAL_CONFIG.value.sortedDataColumnIndices.includes(index);
}

function hasAdditionalSorting(th) {
    if (th.type === 'name' || th === 'name') {
        return FINAL_CONFIG.value.sortedSeriesName;
    }
    if (th.type === 'sum' || th === 'sum') {
        return FINAL_CONFIG.value.sortedSum;
    }
    if (th.type === 'average' || th === 'average') {
        return FINAL_CONFIG.value.sortedAverage;
    }
    if (th.type === 'median' || th === 'median') {
        return FINAL_CONFIG.value.sortedMedian;
    }
    return false;
}

function getArrowOpacity(index, th, order) {
    if (['sum', 'average', 'median'].includes(th.type)) {
        return currentAdditionalSort.value === th.type ? sortOrders.value[th.type] === order ? 1 : 0.3 : 0.3;
    } else {
        return index === currentSortingIndex.value ? sortStates.value[index].state === order ? 1 : 0.3 : 0.3;
    }
}

function resetOnClickOutside() {
    FINAL_CONFIG.value.resetSortOnClickOutside && restoreOrder();
}

const sortStates = computed({
    get: () => {
        return colNames.value.map(_c => {
            return { state: 1 };
        })
    },
    set: (_) => {
        return _;
    }
});

defineExpose({
    generatePdf,
    generateImage,
    generateCsv,
    restoreOrder
})

</script>

<template>
    <div ref="tableContainer" :class="{ 'vue-ui-table-sparkline': true, 'vue-ui-responsive': isResponsive }" style="overflow: hidden" :id="`table_${uid}`" @mouseenter="() => setUserOptionsVisibility(true)" @mouseleave="() => setUserOptionsVisibility(false)">
        
        <div 
            v-if="FINAL_CONFIG.title.text" 
            class="vue-ui-table-sparkline-caption" 
            :style="{ backgroundColor: FINAL_CONFIG.title.backgroundColor }"
        >
            <div class="atom-title" :style="{
                fontSize: `${FINAL_CONFIG.title.fontSize}px`,
                fontWeight: FINAL_CONFIG.title.bold ? 'bold' : 'normal',
                color: FINAL_CONFIG.title.color,
                textAlign: FINAL_CONFIG.title.textAlign,
            }">
                {{ FINAL_CONFIG.title.text }}
            </div>
            <div class="atom-subtitle" v-if="FINAL_CONFIG.title.subtitle.text" :style="{
                fontSize: `${FINAL_CONFIG.title.subtitle.fontSize}px`,
                fontWeight: FINAL_CONFIG.title.subtitle.bold ? 'bold' : 'normal',
                color: FINAL_CONFIG.title.subtitle.color,
                textAlign: FINAL_CONFIG.title.textAlign,
            }">
                {{ FINAL_CONFIG.title.subtitle.text }}
            </div>
        </div>
        <div style="overflow: auto" @pointerleave="selectedSerieIndex = undefined; selectedDataIndex = undefined">
            <table data-cy="vue-data-ui-table-sparkline" class="vue-ui-data-table"
                :style="{ fontFamily: FINAL_CONFIG.fontFamily, position: 'relative' }">


                <thead style="z-index: 1;padding-right:24px">
                    <tr 
                        role="row" 
                        class="vue-ui-data-table__thead-row"                         
                        v-click-outside="resetOnClickOutside" 
                        :style="{
                            backgroundColor: FINAL_CONFIG.thead.backgroundColor,
                            color: FINAL_CONFIG.thead.color
                        }"
                    >
                        <th role="cell" :style="{
                            backgroundColor: FINAL_CONFIG.thead.backgroundColor,
                            border: FINAL_CONFIG.thead.outline,
                            textAlign: FINAL_CONFIG.thead.textAlign,
                            fontWeight: FINAL_CONFIG.thead.bold ? 'bold' : 'normal',
                        }" class="sticky-col-first">
                            <div
                                :style="{
                                    display: 'flex', 
                                    flexDirection: 'row',
                                    alignItems: 'center',
                                    gap: '3px',
                                    justifyContent: FINAL_CONFIG.thead.textAlign,
                                }">
                                <span>{{ FINAL_CONFIG.translations.serie }}</span>

                                <div 
                                    v-if="mutableDataset.length > 1 && FINAL_CONFIG.sortedSeriesName"
                                    :style="{
                                        display: 'flex',
                                        flexDirection: 'row',
                                        alignItems: 'center'
                                    }"
                                >
                                    <button 
                                        class="vue-ui-table-sparkline-sorting-button vue-ui-table-sparkline-sorting-button-down"
                                        @click="orderDatasetByIndex({type: 'name'}, null, -1)"
                                    >
                                        <BaseIcon 
                                            :size="12" 
                                            name="arrowBottom"
                                            :stroke="FINAL_CONFIG.thead.color"
                                            :style="{
                                                opacity: currentAdditionalSort === 'name' ? sortOrders.name === -1 ? 1 : 0.3 : 0.3
                                            }"
                                        />
                                    </button>
                                    <button 
                                        class="vue-ui-table-sparkline-sorting-button vue-ui-table-sparkline-sorting-button-up"
                                        @click="orderDatasetByIndex({type: 'name'}, null, 1)"
                                    >
                                        <BaseIcon 
                                            :size="12" 
                                            name="arrowTop"
                                            :stroke="FINAL_CONFIG.thead.color"
                                            :style="{
                                                opacity: currentAdditionalSort === 'name' ? sortOrders.name === 1 ? 1 : 0.3 : 0.3
                                            }"
                                        />
                                    </button>
                                </div>
                            </div>
                        </th>
                        <th 
                            data-cy="th"
                            role="cell" v-for="(th, i) in colNames" :style="{
                            background: FINAL_CONFIG.thead.backgroundColor,
                            border: FINAL_CONFIG.thead.outline,
                            textAlign: FINAL_CONFIG.thead.textAlign,
                            fontWeight: FINAL_CONFIG.thead.bold ? 'bold' : 'normal',
                            minWidth: i === colNames.length - 1 ? `${FINAL_CONFIG.sparkline.dimensions.width}px` : '48px',
                            paddingRight: i === colNames.length - 1 && FINAL_CONFIG.userOptions.show ? '36px' : '',
                        }" :class="{'sticky-col': i === colNames.length - 1 && FINAL_CONFIG.showSparklines}" 
                        >
                            <div
                                :style="{
                                    display: 'flex', 
                                    flexDirection: 'row',
                                    alignItems: 'center',
                                    gap: '3px',
                                    justifyContent: FINAL_CONFIG.thead.textAlign,
                                }">
                                <span>{{ th.value }}</span>

                                <div
                                    v-if="mutableDataset.length > 1 && (hasSorting(i) || hasAdditionalSorting(th))"
                                    :style="{
                                        display: 'flex',
                                        flexDirection: 'row',
                                        alignItems: 'center'
                                    }"
                                >                                
                                    <button 
                                        class="vue-ui-table-sparkline-sorting-button vue-ui-table-sparkline-sorting-button-down"
                                        @click="() => orderDatasetByIndex(th, i, -1)"
                                    >
                                        <BaseIcon 
                                            :size="12" 
                                            
                                            name="arrowBottom"
                                            :stroke="FINAL_CONFIG.thead.color"
                                            :style="{
                                                opacity: getArrowOpacity(i, th, -1)
                                            }"
                                        />
                                    </button>
                                    <button 
                                        class="vue-ui-table-sparkline-sorting-button vue-ui-table-sparkline-sorting-button-up"
                                        @click="() => orderDatasetByIndex(th, i, 1)"
                                    >
                                        <BaseIcon 
                                            :size="12" 
                                            name="arrowTop"
                                            :stroke="FINAL_CONFIG.thead.color"
                                            :style="{
                                                opacity: getArrowOpacity(i, th, 1)
                                            }"
                                        />
                                    </button>
                                </div>
                            </div>
                            <UserOptions
                                ref="details"
                                :key="`user_option_${step}`"
                                v-if="FINAL_CONFIG.userOptions.show && i === colNames.length - 1 && (keepUserOptionState ? true : userOptionsVisible)"
                                :backgroundColor="FINAL_CONFIG.thead.backgroundColor"
                                :color="FINAL_CONFIG.thead.color"
                                :isPrinting="isPrinting"
                                :isImaging="isImaging"
                                :uid="uid"
                                :hasPdf="FINAL_CONFIG.userOptions.buttons.pdf"
                                :hasXls="FINAL_CONFIG.userOptions.buttons.csv"
                                :hasImg="FINAL_CONFIG.userOptions.buttons.img"
                                :hasFullscreen="FINAL_CONFIG.userOptions.buttons.fullscreen"
                                :isFullscreen="isFullscreen"
                                :titles="{ ...FINAL_CONFIG.userOptions.buttonTitles }"
                                :chartElement="tableContainer"
                                :position="FINAL_CONFIG.userOptions.position"
                                :callbacks="FINAL_CONFIG.userOptions.callbacks"
                                :printScale="FINAL_CONFIG.userOptions.print.scale"
                                @toggleFullscreen="toggleFullscreen"
                                @generatePdf="generatePdf"
                                @generateImage="generateImage"
                                @generateCsv="generateCsv"
                                :style="{
                                    visibility: keepUserOptionState ? userOptionsVisible ? 'visible' : 'hidden' : 'visible'
                                }"
                            >
                                <template #menuIcon="{ isOpen, color }" v-if="$slots.menuIcon">
                                    <slot name="menuIcon" v-bind="{ isOpen, color }"/>
                                </template>
                                <template #optionPdf v-if="$slots.optionPdf">
                                    <slot name="optionPdf" />
                                </template>
                                <template #optionCsv v-if="$slots.optionCsv">
                                    <slot name="optionCsv" />
                                </template>
                                <template #optionImg v-if="$slots.optionImg">
                                    <slot name="optionImg" />
                                </template>
                                <template v-if="$slots.optionFullscreen" template #optionFullscreen="{ toggleFullscreen, isFullscreen }">
                                    <slot name="optionFullscreen" v-bind="{ toggleFullscreen, isFullscreen }"/>
                                </template>
                            </UserOptions>
                        </th>
                    </tr>
                </thead>
                <tbody>
                    <tr 
                        data-cy="tr"
                        role="row" v-for="(tr, i) in mutableDataset" :style="{
                        backgroundColor: FINAL_CONFIG.tbody.backgroundColor,
                        color: FINAL_CONFIG.tbody.color,
                    }" :class="{'vue-ui-data-table__tbody__row' : true, 'vue-ui-data-table__tbody__row-even': i % 2 === 0, 'vue-ui-data-table__tbody__row-odd': i % 2 !== 0}">
                        <td role="cell" :style="{
                            backgroundColor: FINAL_CONFIG.tbody.backgroundColor,
                            border: FINAL_CONFIG.tbody.outline,
                            fontSize: `${FINAL_CONFIG.tbody.fontSize}px`,
                            fontWeight: FINAL_CONFIG.tbody.bold ? 'bold' : 'normal',
                            textAlign: FINAL_CONFIG.tbody.textAlign,
                        }" :data-cell="FINAL_CONFIG.translations.serie" class="vue-ui-data-table__tbody__td sticky-col-first">
                            <div dir="auto"
                                :style="{
                                    display: 'flex', 
                                    flexDirection: 'row',
                                    alignItems: 'center',
                                    gap: '6px',
                                    justifyContent: FINAL_CONFIG.tbody.textAlign
                                }">
                                <span v-if="FINAL_CONFIG.tbody.showColorMarker" :style="{ color: tr.color }">⬤</span>
                                <span>{{ tr.name ?? "-" }}</span>
                            </div>
                        </td>
                        <td dir="auto" role="cell" v-for="(_, j) in maxSeries" ref="TD" :style="{
                            border: FINAL_CONFIG.tbody.outline,
                            fontSize: `${FINAL_CONFIG.tbody.fontSize}px`,
                            fontWeight: FINAL_CONFIG.tbody.bold ? 'bold' : 'normal',
                            textAlign: FINAL_CONFIG.tbody.textAlign,
                            background:
                                selectedDataIndex !== undefined &&
                                    j === selectedDataIndex 
                                    ? FINAL_CONFIG.tbody.selectedColor.useSerieColor ? `${tr.color.length > 7 ? tr.color.slice(0,-2) : tr.color }33` : FINAL_CONFIG.tbody.selectedColor.fallback
                                    : '',
                        }" :data-cell="colNames[j] ? colNames[j].value : ''" class="vue-ui-data-table__tbody__td" @pointerenter="selectedSerieIndex = i; selectedDataIndex = j">
                            {{ [null, undefined].includes(tr.values[j]) ? '-' : applyDataLabel(
                                FINAL_CONFIG.formatter,
                                Number(tr.values[j]),
                                dataLabel({
                                    p: FINAL_CONFIG.prefix,
                                    v: Number(tr.values[j]),
                                    s: FINAL_CONFIG.suffix,
                                    r: FINAL_CONFIG.roundingValues
                                }),
                                { datapoint: tr, seriesIndex: i, datapointIndex: j }
                                )
                            }}
                        </td>
                        <td dir="auto" role="cell" v-if="FINAL_CONFIG.showTotal" :style="{
                            border: FINAL_CONFIG.tbody.outline,
                            fontSize: `${FINAL_CONFIG.tbody.fontSize}px`,
                            fontWeight: FINAL_CONFIG.tbody.bold ? 'bold' : 'normal',
                            textAlign: FINAL_CONFIG.tbody.textAlign,
                        }" :data-cell="FINAL_CONFIG.translations.total" class="vue-ui-data-table__tbody__td">
                            {{ applyDataLabel(
                                FINAL_CONFIG.formatter,
                                tr.sum,
                                dataLabel({
                                    p: FINAL_CONFIG.prefix,
                                    v: tr.sum,
                                    s: FINAL_CONFIG.suffix,
                                    r: FINAL_CONFIG.roundingTotal,
                                }),
                                { datapoint: tr.sum, seriesIndex: i }
                                )
                            }}
                        </td>
                        <td dir="auto" role="cell" v-if="FINAL_CONFIG.showAverage" :style="{
                            border: FINAL_CONFIG.tbody.outline,
                            fontSize: `${FINAL_CONFIG.tbody.fontSize}px`,
                            fontWeight: FINAL_CONFIG.tbody.bold ? 'bold' : 'normal',
                            textAlign: FINAL_CONFIG.tbody.textAlign,
                        }" :data-cell="FINAL_CONFIG.translations.average" class="vue-ui-data-table__tbody__td">
                            {{ applyDataLabel(
                                FINAL_CONFIG.formatter,
                                tr.average,
                                dataLabel({
                                    p: FINAL_CONFIG.prefix,
                                    v: tr.average,
                                    s: FINAL_CONFIG.suffix,
                                    r: FINAL_CONFIG.roundingAverage,
                                }),
                                { datapoint: tr.average, seriesIndex: i }
                                ) 
                            }}
                        </td>
                        <td dir="auto" role="cell" v-if="FINAL_CONFIG.showMedian" :style="{
                            border: FINAL_CONFIG.tbody.outline,
                            fontSize: `${FINAL_CONFIG.tbody.fontSize}px`,
                            fontWeight: FINAL_CONFIG.tbody.bold ? 'bold' : 'normal',
                            textAlign: FINAL_CONFIG.tbody.textAlign,
                        }" :data-cell="FINAL_CONFIG.translations.median" class="vue-ui-data-table__tbody__td">
                            {{ applyDataLabel(
                                FINAL_CONFIG.formatter,
                                tr.median,
                                dataLabel({
                                    p: FINAL_CONFIG.prefix,
                                    v: tr.median,
                                    s: FINAL_CONFIG.suffix,
                                    r: FINAL_CONFIG.roundingMedian,
                                }),
                                { datapoint: tr.median, seriesIndex: i }
                            )}}
                        </td>
                        <td role="cell" v-if="FINAL_CONFIG.showSparklines" :data-cell="FINAL_CONFIG.translations.chart" :style="{
                            border: FINAL_CONFIG.tbody.outline,
                            fontSize: `${FINAL_CONFIG.tbody.fontSize}px`,
                            fontWeight: FINAL_CONFIG.tbody.bold ? 'bold' : 'normal',
                            textAlign: FINAL_CONFIG.tbody.textAlign,
                            backgroundColor: FINAL_CONFIG.tbody.backgroundColor,
                            padding: '0',
                        }" class="vue-ui-data-table__tbody__td sticky-col">
                            <SparkLine 
                                @hoverIndex="({ index }) => hoverSparkline({ dataIndex: index, serieIndex: i })
                                "
                                :height-ratio="FINAL_CONFIG.sparkline.dimensions.heightRatio"
                                :forced-padding="30"
                                :dataset="tr.sparklineDataset" :showInfo="false" :selectedIndex="selectedDataIndex" :config="{
                                type: FINAL_CONFIG.sparkline.type,
                                style: {
                                    backgroundColor: 'transparent',
                                    animation: {
                                        show: FINAL_CONFIG.sparkline.animation.show && !isPrinting && !isImaging,
                                        animationFrames: FINAL_CONFIG.sparkline.animation.animationFrames
                                    },
                                    line: {
                                        color: tr.color,
                                        smooth: FINAL_CONFIG.sparkline.smooth,
                                        strokeWidth: FINAL_CONFIG.sparkline.strokeWidth
                                    },
                                    bar: {
                                        color: tr.color
                                    },
                                    area: {
                                        color: tr.color,
                                        opacity: FINAL_CONFIG.sparkline.showArea ? 16 : 0,
                                        useGradient: FINAL_CONFIG.sparkline.useGradient,
                                    },
                                    verticalIndicator: {
                                        color: tr.color,
                                    },
                                    plot: {
                                        radius: 9,
                                        stroke: FINAL_CONFIG.tbody.backgroundColor,
                                        strokeWidth: 3,
                                    },
                                },
                            }" />
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
        
        <div v-if="$slots.source" ref="source" dir="auto">
            <slot name="source" />
        </div>
    </div>
</template>

<style lang="scss" scoped>
.vue-ui-data-table thead {
    position: sticky;
    top: 0;
    user-select: none;
}

table {
    width: 100%;
    padding: 1rem;
    border-collapse: collapse;
}

caption,
th,
td {
    padding: 0.5rem;
    font-variant-numeric: tabular-nums;
}

th,
td {
    white-space: nowrap;
}

.sticky-col {
    position: -webkit-sticky;
    position: sticky;
    width: 100px;
    min-width: 100px;
    right: 0;
}

.sticky-col-first {
    position: -webkit-sticky;
    position: sticky;
    width: 100px;
    min-width: 100px;
    left: 0;
    z-index: 1;
}

.vue-ui-responsive {
    .sticky-col,
    .sticky-col-first {
        position: initial;
        width: initial;
        min-width: initial;
        left: initial;
        right: initial;
    }
    
    th {
        display: none;
    }

    td {
        display: grid;
        gap: 0.5rem;
        grid-template-columns: repeat(2, 1fr);
        padding: 0.5rem 1rem;
        text-align: left;
    }

    tr {
        outline: v-bind(tdo);
    }

    td:first-child {
        padding-top: 1rem;
    }

    td:last-child {
        padding-bottom: 1rem;
    }

    td::before {
        content: attr(data-cell) ": ";
        font-weight: 700;
        text-transform: capitalize;
    }
}

.vue-ui-table-sparkline-sorting-button {
    cursor: pointer;
    border: none;
    background: transparent;
    display: flex;
    align-items:center;
    justify-content:center;
    height: 16px;
    width: 16px;
    padding: 0;
}

.vue-ui-table-sparkline-sorting-button-down {
    margin-left: 3px;
}
</style>
