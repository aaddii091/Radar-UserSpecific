<template>
  <div class="hello" ref="chartdiv"></div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";
import * as am5 from "@amcharts/amcharts5";
import * as am5xy from "@amcharts/amcharts5/xy";
import * as am5radar from "@amcharts/amcharts5/radar";
import am5themes_Animated from "@amcharts/amcharts5/themes/Animated";
import json from "../event.json";

const chartdiv = ref(null);
let root;
let chart;
const radarData = ref([{}]);
const username = ref("karx");
const userspecificData = ref([{}]);

onMounted(() => {
  root = am5.Root.new(chartdiv.value);
  json.forEach((i) => {
    if (i.value && i.value.searchEntity && i.actor && i.actor.username) {
      radarData.value.push({ name: i.value.searchEntity });
      if (i.actor.username === "karx") {
        userspecificData.value.push({ name: i.value.searchEntity });
      }
    }
  });

  const uniqueNames = radarData.value.reduce((accumulator, currentValue) => {
    const name = currentValue.name;
    const existingItem = accumulator.find((item) => item.name === name);

    if (existingItem) {
      existingItem.value1++;
    } else {
      accumulator.push({ name, value1: 1 });
    }

    return accumulator;
  }, []);
  uniqueNames.shift();
  console.log(uniqueNames);
  const userspecificCount = userspecificData.value.reduce(
    (accumulator, currentValue) => {
      const name = currentValue.name;
      const existingItem = accumulator.find((item) => item.name === name);

      if (existingItem) {
        existingItem.value2++;
      } else {
        accumulator.push({ name, value2: 1 });
      }

      return accumulator;
    },
    []
  );

  console.log(userspecificCount);
  userspecificCount.shift();
  console.log(userspecificCount);

  // Merge the arrays by summing "value1" and "value2"
  const mergedArray = [];

  for (
    let i = 0;
    i < Math.max(userspecificCount.length, uniqueNames.length);
    i++
  ) {
    const item1 = userspecificCount[i] || { value2: 0 };
    const item2 = uniqueNames[i] || { value1: 0 };
    mergedArray.push({
      name: item2.name,
      value1: (item1.value1 || 0) + (item2.value1 || 0),
      value2: (item1.value2 || 0) + (item2.value2 || 0),
    });
  }

  console.log(mergedArray);

  root.setThemes([am5themes_Animated.new(root)]);

  chart = root.container.children.push(
    am5radar.RadarChart.new(root, {
      panX: false,
      panY: false,
      wheelX: "panX",
      wheelY: "zoomX",
      innerRadius: am5.percent(40),
      radius: am5.percent(70),
      arrangeTooltips: false,
    })
  );

  let cursor = chart.set(
    "cursor",
    am5radar.RadarCursor.new(root, {
      behavior: "zoomX",
    })
  );

  cursor.lineY.set("visible", false);

  let xRenderer = am5radar.AxisRendererCircular.new(root, {
    minGridDistance: 30,
  });

  xRenderer.labels.template.setAll({
    textType: "radial",
    radius: 10,
    paddingTop: 0,
    paddingBottom: 0,
    centerY: am5.p50,
    fontSize: "0.8em",
  });

  xRenderer.grid.template.setAll({
    location: 0.5,
    strokeDasharray: [2, 2],
  });

  let xAxis = chart.xAxes.push(
    am5xy.CategoryAxis.new(root, {
      maxDeviation: 0,
      categoryField: "name",
      renderer: xRenderer,
      tooltip: am5.Tooltip.new(root, {}),
    })
  );

  let yRenderer = am5radar.AxisRendererRadial.new(root, {
    minGridDistance: 30,
  });

  yRenderer.grid.template.setAll({
    strokeDasharray: [2, 2],
  });

  let yAxis = chart.yAxes.push(
    am5xy.ValueAxis.new(root, {
      renderer: yRenderer,
    })
  );

  let series1 = chart.series.push(
    am5radar.RadarLineSeries.new(root, {
      name: "Cash held outside",
      xAxis: xAxis,
      yAxis: yAxis,
      valueYField: "value1",
      categoryXField: "name",
    })
  );

  series1.strokes.template.setAll({
    strokeOpacity: 0,
  });

  series1.fills.template.setAll({
    visible: true,
    fillOpacity: 0.5,
  });

  let series2 = chart.series.push(
    am5radar.RadarLineSeries.new(root, {
      name: "Cash held in US",
      xAxis: xAxis,
      yAxis: yAxis,
      valueYField: "value2",
      categoryXField: "name",
      stacked: true,
      tooltip: am5.Tooltip.new(root, {
        labelText: "Outside: {value1}\nInside:{value2}",
      }),
    })
  );

  series2.strokes.template.setAll({
    strokeOpacity: 0,
  });

  series2.fills.template.setAll({
    visible: true,
    fillOpacity: 0.5,
  });

  let legend = chart.radarContainer.children.push(
    am5.Legend.new(root, {
      width: 150,
      centerX: am5.p50,
      centerY: am5.p50,
    })
  );
  legend.data.setAll([series1, series2]);

  series1.data.setAll(mergedArray);
  series2.data.setAll(mergedArray);
  xAxis.data.setAll(mergedArray);

  series1.appear(1000);
  series2.appear(1000);
  chart.appear(1000, 100);
});

onBeforeUnmount(() => {
  if (root) {
    root.dispose();
  }
});
</script>

<style>
.hello {
  width: 100%;
  height: 700px;
  font-size: 8pt;
}
</style>
