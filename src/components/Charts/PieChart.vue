<template>
  <div class="chart" ref="chartRef"></div>
</template>

<script setup lang="ts" name="PieCharts">
import { useChart } from '@/hooks/useChart';
import type { EChartsOption } from 'echarts';
import { ref, type Ref, onMounted } from 'vue';

const chartRef: Ref = ref(null);
const { setOptions } = useChart(chartRef);
let option: EChartsOption;

onMounted(() => {
  init();
});

const init = () => {
  option = {
    title: {
      text: '',
      left: 'center',
      textStyle: {
        color: '#fff',
        fontSize: '0.2rem',
      },
    },
    tooltip: {
      trigger: 'item',
      textStyle: {
        fontSize: '0.2rem',
      },
    },
    // legend: {
    //   orient: 'vertical',
    //   left: 'left',
    // },
    series: [
      {
        name: 'Access From',
        type: 'pie',
        radius: '50%',
        data: [
          { value: 1048, name: '测试1' },
          { value: 735, name: '测试2' },
          { value: 580, name: '测试3' },
          { value: 484, name: '测试4' },
          { value: 300, name: '测试5' },
        ],
        emphasis: {
          itemStyle: {
            shadowBlur: 10,
            shadowOffsetX: 0,
            shadowColor: 'rgba(0, 0, 0, 0.5)',
          },
        },
        label: {
          // fontSize: '0.2rem',
          color: '#fff',
          fontWeight: 'bold',
        },
      },
    ],
    textStyle: {
    },
  };
  
  setOptions(option);
};


</script>

<style scope>
.chart {
  width: 100%;
  height: 40vh;
}
</style>
