<script setup>
import * as echarts from 'echarts/core';
import {TooltipComponent, VisualMapComponent} from 'echarts/components';
import {CanvasRenderer} from 'echarts/renderers';
import {SurfaceChart} from 'echarts-gl/charts';
import {Grid3DComponent} from 'echarts-gl/components';

import {computed, onMounted, ref, watch} from 'vue';
import {NButton, NGi, NGrid, NInput, NPageHeader, NProgress, NSelect, NSpace, NSwitch, useNotification} from 'naive-ui';

import {all, create} from 'mathjs'

const config = {}
const math = create(all, config)
const notification = useNotification()

echarts.use([
  TooltipComponent,
  VisualMapComponent,
  Grid3DComponent,
  SurfaceChart,
  CanvasRenderer
]);
let chartDom = null
let myChart = null
let option = null
onMounted(() => {
  chartDom = document.getElementById('chart');
  myChart = echarts.init(chartDom);
  option = {};
})

function convertChinesePunctuationToEnglish(str) {
  // 定义一个正则表达式匹配中文标点符号
  const chinesePunctuationRegex = /[！，。“”‘’：；？【】（）]/g;

  // 定义一个映射对象，将中文标点符号映射到对应的英文标点符号
  const punctuationMap = {
    '！': '!',
    '，': ',',
    '。': '.',
    '“': '"',
    '”': '"',
    '‘': "'",
    '’': "'",
    '：': ':',
    '；': ';',
    '？': '?',
    '【': '[',
    '】': ']',
    '（': '(',
    '）': ')'
  };

  // 使用replace方法将匹配到的中文标点符号替换为对应的英文标点符号
  return str.replace(chinesePunctuationRegex, function (match) {
    return punctuationMap[match];
  });
}

const ComputeInput = (input) => {
  return math.evaluate(convertChinesePunctuationToEnglish(input))
}

/*
const 初始房价 = ref(5000000)
const 首付率 = ref(0.3)
const 房价增长率 = ref(5)
const 半衰位置 = ref(30)
const 折损率 = ref(0.5)
const 贷款年化利率 = ref(0.05)
const 贷款年限 = ref(30)
const 租售比 = ref(0.005)
const 年租金增长率 = ref(0.01)
*/
//方便下面循环,把上面的变量都放到一个对象里
const inputData = ref({
  初始房价: "5000000",
  首付率: "0.3",
  房价增长率: "5",
  半衰位置: "30",
  折损率: "0.5",
  贷款年化利率: "0.05",
  贷款年限: "30",
  租售比: "0.005",
  年租金增长率: "0.01",
  银行存款利率: "0.03",
})
//从左到右是最小值，最大值，步长，函数
//const xData = ref(Array(4).fill(''))
//const yData = ref(Array(4).fill(''))
const xData = ref(['500000', '50000000', '500000', 'x'])
const yData = ref(['0.2', '0.4', '0.01', 'x'])

/*const linspace = (startValue, stopValue, cardinality) => {
  const arr = []
  const step = (stopValue - startValue) / (cardinality - 1)
  for (let i = 0; i < cardinality; i++) {
    arr.push(startValue + (step * i))
  }
  return arr
}*/
const linspace = (startValue, stopValue, step) => {
  const arr = []
  for (let i = startValue; i < stopValue; i += step) {
    arr.push(i)
  }
  return arr
}
const xFunction = ref('x')
const x = computed(() => {
  return math.compile(xFunction.value)
})
const yFunction = ref('y')
const y = computed(() => {
  return math.compile(yFunction.value)
})

const 月供 = (可变初始房价, 可变贷款年化利率, 可变贷款年限, 可变首付率) => {
  let 月利率 = 可变贷款年化利率 / 12
  return (可变初始房价 * (1 - 可变首付率)) * (月利率 * (1 + 月利率) ** (可变贷款年限 * 12)) / ((1 + 月利率) ** (可变贷款年限 * 12) - 1)
}
const 房屋增值率 = (可变房价增长率, 可变半衰位置, 可变折损率, 当前年份) => {
  return 可变房价增长率 / (1 + (math.e ** (当前年份 - 可变半衰位置))) - 可变折损率
}

const xOption = ref([
  {
    label: '初始房价',
    value: '初始房价'
  },
  {
    label: '首付率',
    value: '首付率'
  },
  {
    label: '房价增长率',
    value: '房价增长率'
  },
  {
    label: '贷款年化利率',
    value: '贷款年化利率'
  },
  {
    label: '贷款年限',
    value: '贷款年限'
  },
  {
    label: '租售比',
    value: '租售比'
  },
  {
    label: '年租金增长率',
    value: '年租金增长率'
  },
  {
    label: '银行存款利率',
    value: '银行存款利率'
  }
])
const xChoice = ref('初始房价')
watch(xChoice, (newValue, oldValue) => {
  yOption.value.forEach((item) => {
    if (item.value === newValue) {
      item.disabled = true
    }
    if (item.value === oldValue) {
      item.disabled = false
    }
  })
})
const yOption = ref([
  {
    label: '初始房价',
    value: '初始房价'
  },
  {
    label: '首付率',
    value: '首付率'
  },
  {
    label: '房价增长率',
    value: '房价增长率'
  },
  {
    label: '贷款年化利率',
    value: '贷款年化利率'
  },
  {
    label: '贷款年限',
    value: '贷款年限'
  },
  {
    label: '租售比',
    value: '租售比'
  },
  {
    label: '年租金增长率',
    value: '年租金增长率'
  },
  {
    label: '银行存款利率',
    value: '银行存款利率'
  }
])
const yChoice = ref('首付率')
watch(yChoice, (newValue, oldValue) => {
  xOption.value.forEach((item) => {
    if (item.value === newValue) {
      item.disabled = true
    }
    if (item.value === oldValue) {
      item.disabled = false
    }
  })
})

const zOption = ref([
  {
    label: '买房总开销',
    value: '买房总开销'
  },
  {
    label: '租房总开销',
    value: '租房总开销'
  },
  {
    label: '买房总开销 - 租房总开销',
    value: '买房总开销 - 租房总开销'
  },
  {
    label: '买房总收益',
    value: '买房总收益'
  },
  {
    label: '租房总收益',
    value: '租房总收益'
  },
  {
    label: '买房总收益 - 租房总收益',
    value: '买房总收益 - 租房总收益'
  },
  {
    label: '买房总资产',
    value: '买房总资产'
  },
  {
    label: '租房总资产',
    value: '租房总资产'
  },
  {
    label: '买房总资产 - 租房总资产',
    value: '买房总资产 - 租房总资产'
  }
])
const zChoice = ref(['买房总资产 - 租房总资产'])
const depositType = ref(false)

let chartSeriesConfig = []
const computeProgress = ref(0)
const computeProgressMax = ref(0)
const computing = ref(false)
const compute = async () => {
  computing.value = true
  computeProgress.value = 0
  let zMax = 0
  let zMin = 0
  chartSeriesConfig = []
  const xF = math.compile(convertChinesePunctuationToEnglish(xData.value[3]))
  const yF = math.compile(convertChinesePunctuationToEnglish(yData.value[3]))
  let xDataToCompute = linspace(ComputeInput(xData.value[0]), ComputeInput(xData.value[1]), ComputeInput(xData.value[2])).map((item) => {
    return xF.evaluate({x: item})
  })
  let yDataToCompute = linspace(ComputeInput(yData.value[0]), ComputeInput(yData.value[1]), ComputeInput(yData.value[2])).map((item) => {
    return yF.evaluate({x: item})
  })
  computeProgressMax.value = xDataToCompute.length * yDataToCompute.length + xDataToCompute.length * yDataToCompute.length * zChoice.value.length
  console.log(computeProgressMax.value)
  let largeCompute = computeProgressMax.value > 10000
  if (largeCompute) {
    notification.create({
      title: '计算量过大，已限制运行速度',
      type: 'warning',
      closable: false,
    })
  }
  notification.create({
    title: '计算中',
    closable: false,
  })
  await new Promise((resolve) => {
    setTimeout(() => {
      resolve()
    }, 200)
  })
  let buyHouseTotalCost = math.ones(xDataToCompute.length, yDataToCompute.length)
  let rentHouseTotalCost = math.ones(xDataToCompute.length, yDataToCompute.length)
  let buyHouseTotalIncome = math.zeros(xDataToCompute.length, yDataToCompute.length)
  let rentHouseTotalIncome = math.ones(xDataToCompute.length, yDataToCompute.length)
  let buyHouseTotalAsset = math.ones(xDataToCompute.length, yDataToCompute.length)
  let rentHouseTotalAsset = math.zeros(xDataToCompute.length, yDataToCompute.length)
  let computeVariable = {
    ...Object.keys(inputData.value).reduce((obj, key) => ({
      ...obj,
      [key]: ComputeInput(inputData.value[key])
    }), {})
  }
  let i = 0
  for (let xIndex = 0; xIndex < xDataToCompute.length; xIndex++) {
    const x = xDataToCompute[xIndex];
    for (let yIndex = 0; yIndex < yDataToCompute.length; yIndex++) {
      const y = yDataToCompute[yIndex];
      if (largeCompute) {
        if (i % 1000 === 0) {
          await new Promise((resolve) => {
            setTimeout(() => {
              resolve();
            }, 1);
          });
        }
        i++
      }

      computeVariable[xChoice.value] = x;
      computeVariable[yChoice.value] = y;

      buyHouseTotalCost.set([xIndex, yIndex], 月供(computeVariable['初始房价'], computeVariable['贷款年化利率'], computeVariable['贷款年限'], computeVariable['首付率']));
      buyHouseTotalAsset.set([xIndex, yIndex], math.range(1, computeVariable['贷款年限'] * 12, true)._data.reduce((sum, current) => {
        return sum * (1 + 房屋增值率(computeVariable['房价增长率'], computeVariable['半衰位置'], computeVariable['折损率'], current) * 0.01);
      }, computeVariable['初始房价']));

      let startRent = computeVariable['初始房价'] * computeVariable['租售比'];
      rentHouseTotalCost.set([xIndex, yIndex], (12 * startRent - 12 * startRent * (1 + computeVariable['年租金增长率']) ** computeVariable['贷款年限']) / (-computeVariable['贷款年化利率']));

      if (!depositType.value) {
        rentHouseTotalIncome.set([xIndex, yIndex], (computeVariable['初始房价'] * computeVariable['租售比'] * (1 + computeVariable['银行存款利率']) ** computeVariable['贷款年限'] +
            月供(computeVariable['初始房价'], computeVariable['贷款年化利率'], computeVariable['贷款年限'], computeVariable['首付率']) *
            ((12 * computeVariable['贷款年限'] + 1) / 2) *
            (12 * computeVariable['贷款年限']) * (computeVariable['银行存款利率'] / 12)
        ));
      } else {
        rentHouseTotalIncome.set([xIndex, yIndex], computeVariable['初始房价'] * computeVariable['租售比'] * (1 + computeVariable['银行存款利率']) ** computeVariable['贷款年限']);
      }

      computeProgress.value += 1;
    }
  }
  let first = true
  zChoice.value.forEach((z) => {
    let t = []
    let tem = null
    switch (z) {
      case '买房总开销':
        yDataToCompute.forEach((y, yIndex) => {
          xDataToCompute.forEach((x, xIndex) => {
            t.push([x, y, buyHouseTotalCost.get([xIndex, yIndex])])
            computeProgress.value += 1
          })
        })
        if (first) {
          zMax = math.max(buyHouseTotalCost)
          zMin = math.min(buyHouseTotalCost)
          first = false
        } else {
          zMax = math.max(buyHouseTotalCost) > zMax ? math.max(buyHouseTotalCost) : zMax
          zMin = math.min(buyHouseTotalCost) < zMin ? math.min(buyHouseTotalCost) : zMin
        }
        chartSeriesConfig.push({
          type: 'surface',
          data: t,
          name: '买房总开销'
        })
        break
      case '租房总开销':
        yDataToCompute.forEach((y, yIndex) => {
          xDataToCompute.forEach((x, xIndex) => {
            t.push([x, y, rentHouseTotalCost.get([xIndex, yIndex])])
            computeProgress.value += 1
          })
        })
        if (first) {
          zMax = math.max(rentHouseTotalCost)
          zMin = math.min(rentHouseTotalCost)
          first = false
        } else {
          zMax = math.max(rentHouseTotalCost) > zMax ? math.max(rentHouseTotalCost) : zMax
          zMin = math.min(rentHouseTotalCost) < zMin ? math.min(rentHouseTotalCost) : zMin
        }
        chartSeriesConfig.push({
          type: 'surface',
          data: t,
          name: "租房总开销"
        })
        break
      case '买房总开销 - 租房总开销':
        tem = math.subtract(buyHouseTotalCost, rentHouseTotalCost)
        yDataToCompute.forEach((y, yIndex) => {
          xDataToCompute.forEach((x, xIndex) => {
            t.push([x, y, tem.get([xIndex, yIndex])])
            computeProgress.value += 1
          })
        })
        if (first) {
          zMax = math.max(tem)
          zMin = math.min(tem)
          first = false
        } else {
          zMax = math.max(tem) > zMax ? math.max(tem) : zMax
          zMin = math.min(tem) < zMin ? math.min(tem) : zMin
        }
        chartSeriesConfig.push({
          type: 'surface',
          data: t,
          name: "买房总开销 - 租房总开销"
        })
        break
      case '买房总收益':
        yDataToCompute.forEach((y, yIndex) => {
          xDataToCompute.forEach((x, xIndex) => {
            t.push([x, y, buyHouseTotalIncome.get([xIndex, yIndex])])
            computeProgress.value += 1
          })
        })
        if (first) {
          zMax = math.max(buyHouseTotalIncome)
          zMin = math.min(buyHouseTotalIncome)
          first = false
        } else {
          zMax = math.max(buyHouseTotalIncome) > zMax ? math.max(buyHouseTotalIncome) : zMax
          zMin = math.min(buyHouseTotalIncome) < zMin ? math.min(buyHouseTotalIncome) : zMin
        }
        chartSeriesConfig.push({
          type: 'surface',
          data: t,
          name: "买房总收益"
        })
        break
      case '租房总收益':
        yDataToCompute.forEach((y, yIndex) => {
          xDataToCompute.forEach((x, xIndex) => {
            t.push([x, y, rentHouseTotalIncome.get([xIndex, yIndex])])
            computeProgress.value += 1
          })
        })
        if (first) {
          zMax = math.max(rentHouseTotalIncome)
          zMin = math.min(rentHouseTotalIncome)
          first = false
        } else {
          zMax = math.max(rentHouseTotalIncome) > zMax ? math.max(rentHouseTotalIncome) : zMax
          zMin = math.min(rentHouseTotalIncome) < zMin ? math.min(rentHouseTotalIncome) : zMin
        }
        chartSeriesConfig.push({
          type: 'surface',
          data: t,
          name: "租房总收益"
        })
        break
      case '买房总收益 - 租房总收益':
        tem = math.subtract(buyHouseTotalIncome, rentHouseTotalIncome)
        yDataToCompute.forEach((y, yIndex) => {
          xDataToCompute.forEach((x, xIndex) => {
            t.push([x, y, tem.get([xIndex, yIndex])])
            computeProgress.value += 1
          })
        })
        if (first) {
          zMax = math.max(tem)
          zMin = math.min(tem)
          first = false
        } else {
          zMax = math.max(tem) > zMax ? math.max(tem) : zMax
          zMin = math.min(tem) < zMin ? math.min(tem) : zMin
        }
        chartSeriesConfig.push({
          type: 'surface',
          data: t,
          name: "买房总收益 - 租房总收益"
        })
        break
      case '买房总资产':
        tem = math.add(buyHouseTotalIncome, buyHouseTotalAsset)
        tem = math.subtract(tem, buyHouseTotalCost)
        yDataToCompute.forEach((y, yIndex) => {
          xDataToCompute.forEach((x, xIndex) => {
            t.push([x, y, tem.get([xIndex, yIndex])])
            computeProgress.value += 1
          })
        })
        if (first) {
          zMax = math.max(tem)
          zMin = math.min(tem)
          first = false
        } else {
          zMax = math.max(tem) > zMax ? math.max(tem) : zMax
          zMin = math.min(tem) < zMin ? math.min(tem) : zMin
        }
        chartSeriesConfig.push({
          type: 'surface',
          data: t,
          name: "买房总资产"
        })
        break
      case '租房总资产':
        tem = math.add(rentHouseTotalIncome, rentHouseTotalAsset)
        tem = math.subtract(tem, rentHouseTotalCost)
        yDataToCompute.forEach((y, yIndex) => {
          xDataToCompute.forEach((x, xIndex) => {
            t.push([x, y, tem.get([xIndex, yIndex])])
            computeProgress.value += 1
          })
        })
        if (first) {
          zMax = math.max(tem)
          zMin = math.min(tem)
          first = false
        } else {
          zMax = math.max(tem) > zMax ? math.max(tem) : zMax
          zMin = math.min(tem) < zMin ? math.min(tem) : zMin
        }
        chartSeriesConfig.push({
          type: 'surface',
          data: t,
          name: "租房总资产"
        })
        break
      case '买房总资产 - 租房总资产':
        tem = math.add(buyHouseTotalIncome, buyHouseTotalAsset)
        tem = math.subtract(tem, buyHouseTotalCost)
        let tem2 = math.add(rentHouseTotalIncome, rentHouseTotalAsset)
        tem2 = math.subtract(tem2, rentHouseTotalCost)
        tem = math.subtract(tem, tem2)
        yDataToCompute.forEach((y, yIndex) => {
          xDataToCompute.forEach((x, xIndex) => {
            t.push([x, y, tem.get([xIndex, yIndex])])
            computeProgress.value += 1
          })
        })
        if (first) {
          zMax = math.max(tem)
          zMin = math.min(tem)
          first = false
        } else {
          zMax = math.max(tem) > zMax ? math.max(tem) : zMax
          zMin = math.min(tem) < zMin ? math.min(tem) : zMin
        }
        chartSeriesConfig.push({
          type: 'surface',
          data: t,
          name: "买房总资产 - 租房总资产",
          shading: 'color'
        })
        break
    }
  })
  option = {
    tooltip: {},
    backgroundColor: '#fff',
    visualMap: {
      show: true,
      dimension: 2,
      min: zMin,
      max: zMax,
      inRange: {
        color: [
          '#313695',
          '#4575b4',
          '#74add1',
          '#abd9e9',
          '#e0f3f8',
          '#ffffbf',
          '#fee090',
          '#fdae61',
          '#f46d43',
          '#d73027',
          '#a50026'
        ],
      }
    },
    xAxis3D: {
      type: 'value',
      name: xChoice.value
    },
    yAxis3D: {
      type: 'value',
      name: yChoice.value
    },
    zAxis3D: {
      type: 'value'
    },
    grid3D: {
      light: {
        main: {
          shadow: false
        },
        ambient: {
          intensity: 0.3
        }
      }
    },
    series: chartSeriesConfig
  };
  console.log(option)
  myChart.setOption(option);
  notification.destroyAll()
  computing.value = false
}

</script>

<template>
  <div id="main">
  <NSpace style="margin: auto" vertical>
    <NPageHeader>
      <template #title>
        <a href="./1doc.html">点击前往文档</a>
      </template>
    </NPageHeader>
    <NSpace vertical>
      <div>
        x轴：
        <NSelect v-model:value="xChoice" :options="xOption"/>
      </div>
      <div>
        y轴：
        <NSelect v-model:value="yChoice" :options="yOption"/>
      </div>
      <div>
        z轴：
        <NSelect v-model:value="zChoice" :options="zOption" multiple/>
      </div>
      <div>
        存款收入
        <NSwitch v-model:value="depositType">
          <template #checked>仅计算初始资金的收入</template>
          <template #unchecked>按零存整取计算</template>
        </NSwitch>
      </div>
    </NSpace>
    <NGrid cols="1 s:2 m:2 l:3 xl:3 2xl:4" responsive="screen" x-gap="12">
      <template v-for="item in xOption">
        <NGi v-if="item.value !== xChoice && item.value !== yChoice" :id="item.value">
          {{ item.label }}:
          <NInput v-model:value="inputData[item.value]"/>
        </NGi>
        <NGi v-else-if="item.value === xChoice" :id="item.value">
          {{ item.label }}:
          <NInput v-model:value="xData[0]"/>
          <NInput v-model:value="xData[1]"/>
          <NInput v-model:value="xData[2]"/>
          <NInput v-model:value="xData[3]"/>
        </NGi>
        <NGi v-else-if="item.value === yChoice" :id="item.value">
          {{ item.label }}:
          <NInput v-model:value="yData[0]"/>
          <NInput v-model:value="yData[1]"/>
          <NInput v-model:value="yData[2]"/>
          <NInput v-model:value="yData[3]"/>
        </NGi>
      </template>
    </NGrid>
    <NSpace style="width: 100%" vertical>
      <NSpace justify="center" style="width: 100%">
        <NButton :loading="computing" type="primary" @click="compute">计算</NButton>
      </NSpace>
      <NProgress :percentage="Number((computeProgress / computeProgressMax * 100).toFixed(2))"/>
    </NSpace>
  </NSpace>
  <div id="chart" style="width: 100%;height: 100%"></div>
  </div>
</template>

<style scoped>
@media (min-width: 1024px) {
  #main {
    display: grid;
    grid-template-columns: 1fr 2fr;
    padding: 0 2rem;
  }
}

</style>