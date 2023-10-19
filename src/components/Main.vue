<script setup>
import * as echarts from 'echarts/core';
import { TooltipComponent, VisualMapComponent } from 'echarts/components';
import { CanvasRenderer } from 'echarts/renderers';
import { SurfaceChart } from 'echarts-gl/charts';
import { Grid3DComponent } from 'echarts-gl/components';

echarts.use([
  TooltipComponent,
  VisualMapComponent,
  Grid3DComponent,
  SurfaceChart,
  CanvasRenderer
]);

const chartDom = document.getElementById('chart');
const myChart = echarts.init(chartDom);
let option;



</script>

<template>

  <div id="chart" style="width: 600px;height:400px;"></div>
</template>

<style scoped>

</style>