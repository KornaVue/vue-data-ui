<script setup>
import { ref, computed, onMounted } from "vue";
import { useNestedProp } from "../useNestedProp";
import { dataLabel } from "../lib";
import { useConfig } from "../useConfig";

const { vue_ui_kpi: DEFAULT_CONFIG } = useConfig();

const props = defineProps({
    config: {
        type: Object,
        default() {
            return {}
        }
    },
    dataset: {
        type: Number,
        default: 0
    },
});

const FINAL_CONFIG = computed(() => {
    return useNestedProp({
        userConfig: props.config,
        defaultConfig: DEFAULT_CONFIG
    })
});

const formattedValue = ref(typeof props.dataset === 'number' ? props.dataset : props.dataset);
const displayedValue = ref(FINAL_CONFIG.value.useAnimation ? FINAL_CONFIG.value.animationValueStart : formattedValue.value );

onMounted(() => {
    const chunks = FINAL_CONFIG.value.animationFrames;
    const chunk = props.dataset / chunks;

    function animate() {
        displayedValue.value += chunk;
        if (displayedValue.value < props.dataset) {
            requestAnimationFrame(animate)
        } else {
            displayedValue.value = props.dataset;
        }
    }

    if (FINAL_CONFIG.value.useAnimation) {
        displayedValue.value = 0;
        animate()
    }
});

</script>

<template>
    <div :class="`vue-ui-kpi ${FINAL_CONFIG.layoutClass}`" :style="`background:${FINAL_CONFIG.backgroundColor}; ${FINAL_CONFIG.layoutCss}`">
        <div :class="`vue-ui-kpi-title ${FINAL_CONFIG.titleClass}`" :style="`font-family: ${FINAL_CONFIG.fontFamily}; font-size:${FINAL_CONFIG.titleFontSize}px; color:${FINAL_CONFIG.titleColor}; font-weight:${FINAL_CONFIG.titleBold ? 'bold' : 'normal'}; ${FINAL_CONFIG.titleCss}`">
            <slot name="title" :comment="dataset"></slot>
            {{ FINAL_CONFIG.title }}
        </div>
        <slot name="comment-before" :comment="dataset"></slot>
        <div :class="`vue-ui-kpi-value ${FINAL_CONFIG.valueClass}`" :style="`font-family: ${FINAL_CONFIG.fontFamily}; font-size:${FINAL_CONFIG.valueFontSize}px; color:${FINAL_CONFIG.valueColor}; font-weight:${FINAL_CONFIG.valueBold ? 'bold': 'normal'}; ${FINAL_CONFIG.valueCss}`">
            <slot name="value" :comment="dataset"></slot>
            {{ dataLabel({ p: FINAL_CONFIG.prefix, v: displayedValue, s: FINAL_CONFIG.suffix, r: FINAL_CONFIG.valueRounding }) }}
        </div>
        <slot name="comment-after" :comment="dataset"></slot>
    </div>
</template>