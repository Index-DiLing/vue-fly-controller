<template>
  <VChart ref="chartRef" class="chart" :option="option"/>
</template>

<script setup>
import {use} from "echarts/core";
import {CanvasRenderer} from "echarts/renderers";
import {LineChart} from "echarts/charts";
import {GridComponent, LegendComponent, TitleComponent, TooltipComponent} from "echarts/components";
import VChart, {THEME_KEY} from "vue-echarts";
import {provide, ref, watch, computed, shallowRef} from "vue";


use([CanvasRenderer, LineChart, TitleComponent, TooltipComponent, LegendComponent, GridComponent]);

provide(THEME_KEY, "dark");
const d = defineProps({
  data: {
    type: Array
  },
  title: {
    type: String
  },
  max: {
    type: Number
  },
  min: {
    type: Number
  },
  subtitle: {
    type: Array
  },
  interval:{
    type: Number
  }
});

// 计算 series 配置
const series = [];
if (d.data && d.data[0] !== undefined && typeof d.data[0] === 'object' && Array.isArray(d.data[0])) {
  console.log("是数组")
  // 如果第一个元素不是数字，则认为是多个数据集
  d.data.forEach((item) => {
    series.push({
      data: item,
      type: 'line',
      showSymbol: false,
      // sampling: 'lttb', // 使用采样减少渲染点数
    })
  })
} else {
  // 单个数据集
  series.push({
    data: d.data,
    type: 'line',
    showSymbol: false,
    // sampling: 'lttb',
  })
}

const option = {
  title: {
    text: d.title ? d.title : 'Referer of a Website',
    subtext: d.subtitle ? d.subtitle : 'Fake Data',
    left: 'center'
  },
  tooltip: {
    trigger: 'axis',
    axisPointer: {
      type: 'line'
    }
  },
  xAxis: {
    type: 'category',
    show: false
  },
  yAxis: {
    max: d.max,
    min: d.min,
    interval: d.interval,
    axisLine: {
      show: false
    }
  },
  series: series,
};
</script>

<style scoped>
.chart {
  padding: 0.2%;
  width: 48%;
  height: 50%;
}
</style>